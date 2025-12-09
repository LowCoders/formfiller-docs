# Multisite Management

FormFiller supports multi-tenant operation, where a single application instance serves multiple, isolated sites.

## Site Identification

### Subdomain Based

```
https://site-a.formfiller.com  →  Site A
https://site-b.formfiller.com  →  Site B
```

```typescript
// utils/siteDetection.ts
export function detectSiteFromUrl(): string | null {
  const hostname = window.location.hostname;
  const parts = hostname.split('.');
  
  if (parts.length >= 3) {
    return parts[0]; // subdomain = site slug
  }
  
  return null; // Main domain, no site
}
```

### Path Based

```
https://formfiller.com/site-a/...  →  Site A
https://formfiller.com/site-b/...  →  Site B
```

## SiteContext

### Context Definition

```typescript
interface Site {
  id: string;
  name: string;
  slug: string;
  settings: SiteSettings;
  branding?: SiteBranding;
}

interface SiteContextType {
  currentSite: Site | null;
  availableSites: Site[];
  isLoading: boolean;
  switchSite: (siteId: string) => Promise<void>;
  refreshSite: () => Promise<void>;
}

const SiteContext = createContext<SiteContextType | null>(null);
```

### Hook Usage

```typescript
import { useSite } from '../contexts/SiteContext';

function MyComponent() {
  const { currentSite, availableSites, switchSite } = useSite();

  return (
    <div>
      <h1>{currentSite?.name}</h1>
      <SiteSwitcher sites={availableSites} onSwitch={switchSite} />
    </div>
  );
}
```

## API Calls in Site Context

### Site Header

Every API call includes the current site identifier:

```typescript
// api/client.ts
apiClient.interceptors.request.use(config => {
  const { currentSite } = useSiteStore.getState();
  
  if (currentSite) {
    config.headers['X-Site-ID'] = currentSite.id;
  }
  
  return config;
});
```

## Site Switcher Component

```typescript
function SiteSwitcher() {
  const { currentSite, availableSites, switchSite } = useSite();
  const { hasPermission } = usePermissions();

  // Only if user has access to multiple sites
  if (availableSites.length <= 1) {
    return null;
  }

  return (
    <DropDownButton
      text={currentSite?.name || 'Select site'}
      icon="globe"
      items={availableSites}
      keyExpr="id"
      displayExpr="name"
      onItemClick={e => switchSite(e.itemData.id)}
    />
  );
}
```

## Site Branding

### Branding Configuration

```typescript
interface SiteBranding {
  logo?: string;
  favicon?: string;
  primaryColor?: string;
  secondaryColor?: string;
  customCss?: string;
}
```

### Branding Application

```typescript
function BrandingProvider({ children }: { children: React.ReactNode }) {
  const { currentSite } = useSite();

  useEffect(() => {
    if (currentSite?.branding) {
      const { branding } = currentSite;

      // Favicon
      if (branding.favicon) {
        document.querySelector('link[rel="icon"]')?.setAttribute('href', branding.favicon);
      }

      // CSS variables
      if (branding.primaryColor) {
        document.documentElement.style.setProperty('--primary-color', branding.primaryColor);
      }

      // Custom CSS
      if (branding.customCss) {
        const style = document.createElement('style');
        style.textContent = branding.customCss;
        document.head.appendChild(style);
      }
    }
  }, [currentSite]);

  return <>{children}</>;
}
```

## User Invitation to Site

```typescript
function InviteUserToSite() {
  const { currentSite } = useSite();
  const [email, setEmail] = useState('');
  const [role, setRole] = useState('viewer');

  const handleInvite = async () => {
    await siteService.inviteUser(currentSite.id, {
      email,
      role
    });
    notify('Invitation sent', 'success');
  };

  return (
    <Form>
      <SimpleItem 
        dataField="email" 
        label={{ text: 'Email address' }}
        editorOptions={{ value: email, onValueChanged: e => setEmail(e.value) }}
      />
      <SimpleItem
        dataField="role"
        label={{ text: 'Role' }}
        editorType="dxSelectBox"
        editorOptions={{
          items: availableRoles,
          value: role,
          onValueChanged: e => setRole(e.value)
        }}
      />
      <ButtonItem>
        <ButtonOptions text="Invite" onClick={handleInvite} />
      </ButtonItem>
    </Form>
  );
}
```

## Best Practices

### 1. Site Context Always Available

```typescript
// Ensure SiteProvider is at the top of component tree
<AuthProvider>
  <SiteProvider>
    <PermissionProvider>
      <App />
    </PermissionProvider>
  </SiteProvider>
</AuthProvider>
```

### 2. Handle Site Switching

```typescript
// Clear cache on site switch
const switchSite = async (siteId: string) => {
  // Clear cache
  queryClient.clear();
  
  // Navigate to new site
  // ...
};
```

### 3. Error Handling Without Site

```typescript
function RequireSite({ children }: { children: React.ReactNode }) {
  const { currentSite, isLoading } = useSite();

  if (isLoading) return <Loading />;
  
  if (!currentSite) {
    return <SiteSelectionPage />;
  }

  return <>{children}</>;
}
```

