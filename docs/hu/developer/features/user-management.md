# Felhasználó Kezelés

A FormFiller beépített felhasználó kezelési funkciókat biztosít: regisztráció, bejelentkezés, profil kezelés.

## Autentikációs Módok

### JWT Token Alapú

Az alapértelmezett autentikációs mód JWT (JSON Web Token) használatával:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Login     │────▶│   Backend   │────▶│ JWT Token   │
│   Form      │     │  /auth/login│     │  generálás  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┘
                    ▼
              ┌─────────────┐
              │ localStorage│  Token tárolás
              │   token     │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │  Minden API │  Authorization: Bearer <token>
              │   hívás     │
              └─────────────┘
```

### Google OAuth

OAuth 2.0 bejelentkezés Google fiókkal:

```typescript
// Login gomb
<Button onClick={() => window.location.href = `${API_URL}/auth/google`}>
  Bejelentkezés Google-lel
</Button>
```

## AuthContext

A felhasználói állapot kezelése React Context-tel:

```typescript
import { useAuth } from '../contexts/AuthContext';

function MyComponent() {
  const { 
    user,           // Bejelentkezett felhasználó
    isAuthenticated,// Boolean - be van-e jelentkezve
    isLoading,      // Boolean - töltés alatt
    login,          // Bejelentkezés függvény
    logout,         // Kijelentkezés függvény
    updateProfile   // Profil frissítés
  } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }

  return <div>Üdv, {user.name}!</div>;
}
```

### AuthProvider

Az alkalmazás gyökerében:

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

## Bejelentkezési Folyamat

### Login Oldal

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
      // Sikeres - AuthContext átirányít
    } catch (err) {
      setError('Hibás email vagy jelszó');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <TextBox value={email} onValueChanged={e => setEmail(e.value)} />
      <TextBox mode="password" value={password} onValueChanged={e => setPassword(e.value)} />
      <Button type="submit" text="Bejelentkezés" />
      {error && <div className="error">{error}</div>}
    </form>
  );
}
```

### Védett Útvonalak

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

// Használat
<Route 
  path="/form/:configId" 
  element={
    <ProtectedRoute>
      <FormPage />
    </ProtectedRoute>
  } 
/>
```

## Regisztráció

### Regisztrációs Űrlap

```typescript
function RegisterPage() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    passwordConfirm: ''
  });

  const handleSubmit = async () => {
    if (formData.password !== formData.passwordConfirm) {
      setError('A jelszavak nem egyeznek');
      return;
    }

    await authService.register(formData);
    // Átirányítás login oldalra vagy automatikus bejelentkezés
  };
}
```

### Multisite Regisztráció

Ha a multisite mód engedélyezett, a regisztráció site létrehozással is járhat:

```typescript
const [formData, setFormData] = useState({
  name: '',
  email: '',
  password: '',
  siteName: '',      // Új site neve
  siteSlug: ''       // URL-ben megjelenő azonosító
});
```

## Profil Kezelés

### Profil Oldal

```typescript
function ProfilePage() {
  const { user, updateProfile } = useAuth();
  const [formData, setFormData] = useState({
    name: user?.name || '',
    email: user?.email || ''
  });

  const handleSave = async () => {
    await updateProfile(formData);
    notify('Profil frissítve', 'success');
  };

  return (
    <Form formData={formData} onFieldDataChanged={handleFieldChange}>
      <SimpleItem dataField="name" label={{ text: 'Név' }} />
      <SimpleItem dataField="email" label={{ text: 'Email' }} />
      <ButtonItem>
        <ButtonOptions text="Mentés" onClick={handleSave} />
      </ButtonItem>
    </Form>
  );
}
```

### Jelszó Módosítás

```typescript
const handlePasswordChange = async () => {
  await authService.changePassword({
    currentPassword,
    newPassword,
    confirmPassword
  });
  
  // Kijelentkeztetés - újra be kell jelentkezni
  logout();
  notify('Jelszó módosítva. Jelentkezz be újra.', 'info');
};
```

## Token Kezelés

### Token Tárolás

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

### Token Frissítés

```typescript
// Automatikus token frissítés lejárat előtt
useEffect(() => {
  const interval = setInterval(async () => {
    const token = authService.getToken();
    if (token && isTokenExpiringSoon(token)) {
      const newToken = await authService.refreshToken();
      authService.setToken(newToken);
    }
  }, 60000); // Percenként ellenőrzés

  return () => clearInterval(interval);
}, []);
```

### API Hívások Autentikációval

```typescript
// api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: API_URL
});

// Request interceptor - token hozzáadása
apiClient.interceptors.request.use(config => {
  const token = authService.getToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - 401 kezelés
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

## Felhasználói Adatok

### User Objektum

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  roles: string[];        // Szerepkörök
  permissions: string[];  // Engedélyek
  sites: Site[];          // Elérhető site-ok (multisite)
  currentSite?: Site;     // Aktuális site
  preferences: {
    language: string;
    theme: 'light' | 'dark';
    notifications: boolean;
  };
}
```

### Felhasználó Lekérése

```typescript
// AuthContext inicializálásakor
useEffect(() => {
  const initAuth = async () => {
    const token = authService.getToken();
    if (token) {
      try {
        const user = await authService.getCurrentUser();
        setUser(user);
        setIsAuthenticated(true);
      } catch {
        authService.removeToken();
      }
    }
    setIsLoading(false);
  };

  initAuth();
}, []);
```

## Biztonsági Megfontolások

### Token Biztonság

- Token csak HTTPS-en keresztül
- HttpOnly cookie használata (opcionális, biztonságosabb)
- Token lejárat (alapértelmezett: 24 óra)
- Refresh token rotáció

### Védelem

- CSRF védelem
- XSS védelem (React automatikus escape)
- Rate limiting a backend-en
- Jelszó erősség validáció

### Kijelentkezés

```typescript
const logout = async () => {
  try {
    await authService.logout(); // Backend-et is értesítjük
  } finally {
    authService.removeToken();
    setUser(null);
    setIsAuthenticated(false);
    navigate('/login');
  }
};
```

