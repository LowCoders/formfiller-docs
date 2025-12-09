# User Management

FormFiller provides built-in user management features: registration, login, profile management.

## Authentication Modes

### JWT Token Based

The default authentication mode uses JWT (JSON Web Token):

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Login     │────▶│   Backend   │────▶│ JWT Token   │
│   Form      │     │  /auth/login│     │  generation │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┘
                    ▼
              ┌─────────────┐
              │ localStorage│  Token storage
              │   token     │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │  Every API  │  Authorization: Bearer <token>
              │   call      │
              └─────────────┘
```

### Google OAuth

OAuth 2.0 login with Google account:

```typescript
// Login button
<Button onClick={() => window.location.href = `${API_URL}/auth/google`}>
  Login with Google
</Button>
```

## AuthContext

User state management with React Context:

```typescript
import { useAuth } from '../contexts/AuthContext';

function MyComponent() {
  const { 
    user,           // Logged in user
    isAuthenticated,// Boolean - is logged in
    isLoading,      // Boolean - loading
    login,          // Login function
    logout,         // Logout function
    updateProfile   // Profile update
  } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }

  return <div>Welcome, {user.name}!</div>;
}
```

### AuthProvider

At application root:

```typescript
// App.tsx
import { AuthProvider } from './contexts/AuthContext';

function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          {/* ... */}
        </Routes>
      </Router>
    </AuthProvider>
  );
}
```

## Login Process

### Login Page

```typescript
// pages/login/LoginPage.tsx
function LoginPage() {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login(email, password);
      // Success - AuthContext redirects
    } catch (err) {
      setError('Invalid email or password');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <TextBox value={email} onValueChanged={e => setEmail(e.value)} />
      <TextBox mode="password" value={password} onValueChanged={e => setPassword(e.value)} />
      <Button type="submit" text="Login" />
      {error && <div className="error">{error}</div>}
    </form>
  );
}
```

### Protected Routes

```typescript
// components/ProtectedRoute.tsx
function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
}

// Usage
<Route 
  path="/form/:configId" 
  element={
    <ProtectedRoute>
      <FormPage />
    </ProtectedRoute>
  } 
/>
```

## Profile Management

### Profile Page

```typescript
function ProfilePage() {
  const { user, updateProfile } = useAuth();
  const [formData, setFormData] = useState({
    name: user?.name || '',
    email: user?.email || ''
  });

  const handleSave = async () => {
    await updateProfile(formData);
    notify('Profile updated', 'success');
  };

  return (
    <Form formData={formData} onFieldDataChanged={handleFieldChange}>
      <SimpleItem dataField="name" label={{ text: 'Name' }} />
      <SimpleItem dataField="email" label={{ text: 'Email' }} />
      <ButtonItem>
        <ButtonOptions text="Save" onClick={handleSave} />
      </ButtonItem>
    </Form>
  );
}
```

### Password Change

```typescript
const handlePasswordChange = async () => {
  await authService.changePassword({
    currentPassword,
    newPassword,
    confirmPassword
  });
  
  // Logout - need to login again
  logout();
  notify('Password changed. Please login again.', 'info');
};
```

## Token Management

### Token Storage

```typescript
// services/authService.ts
const TOKEN_KEY = 'formfiller_token';

export const authService = {
  setToken(token: string) {
    localStorage.setItem(TOKEN_KEY, token);
  },

  getToken(): string | null {
    return localStorage.getItem(TOKEN_KEY);
  },

  removeToken() {
    localStorage.removeItem(TOKEN_KEY);
  }
};
```

### API Calls with Authentication

```typescript
// api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: API_URL
});

// Request interceptor - add token
apiClient.interceptors.request.use(config => {
  const token = authService.getToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - handle 401
apiClient.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      authService.removeToken();
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

## User Data

### User Object

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  roles: string[];        // Roles
  permissions: string[];  // Permissions
  sites: Site[];          // Available sites (multisite)
  currentSite?: Site;     // Current site
  preferences: {
    language: string;
    theme: 'light' | 'dark';
    notifications: boolean;
  };
}
```

## Security Considerations

### Token Security

- Token only over HTTPS
- HttpOnly cookie usage (optional, more secure)
- Token expiration (default: 24 hours)
- Refresh token rotation

### Protection

- CSRF protection
- XSS protection (React automatic escape)
- Rate limiting on backend
- Password strength validation

### Logout

```typescript
const logout = async () => {
  try {
    await authService.logout(); // Notify backend too
  } finally {
    authService.removeToken();
    setUser(null);
    setIsAuthenticated(false);
    navigate('/login');
  }
};
```

