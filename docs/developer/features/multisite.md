# Multisite Kezelés

A FormFiller támogatja a több bérlős (multi-tenant) működést, ahol egyetlen alkalmazás példány több, egymástól elkülönített site-ot szolgál ki.

## Multisite Architektúra a Frontend-en

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND ALKALMAZÁS                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                  SITE CONTEXT                        │   │
│  │   currentSite │ availableSites │ switchSite         │   │
│  └───────────────────────────┬─────────────────────────┘   │
│                              │                              │
│          ┌───────────────────┼───────────────────┐         │
│          ▼                   ▼                   ▼         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │   Site A    │    │   Site B    │    │   Site C    │    │
│  │  Adatok     │    │   Adatok    │    │   Adatok    │    │
│  │  Configok   │    │   Configok  │    │   Configok  │    │
│  │  Users      │    │   Users     │    │   Users     │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Site Azonosítás

### Subdomain Alapú

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
  
  return null; // Fő domain, nincs site
}
```

### Path Alapú

```
https://formfiller.com/site-a/...  →  Site A
https://formfiller.com/site-b/...  →  Site B
```

```typescript
export function detectSiteFromPath(): string | null {
  const pathParts = window.location.pathname.split('/');
  
  if (pathParts[1] && isValidSiteSlug(pathParts[1])) {
    return pathParts[1];
  }
  
  return null;
}
```

## SiteContext

### Context Definíció

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

### SiteProvider

```typescript
export function SiteProvider({ children }: { children: React.ReactNode }) {
  const [currentSite, setCurrentSite] = useState<Site | null>(null);
  const [availableSites, setAvailableSites] = useState<Site[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const initSite = async () => {
      const siteSlug = detectSiteFromUrl() || detectSiteFromPath();
      
      if (siteSlug) {
        const site = await siteService.getBySlug(siteSlug);
        setCurrentSite(site);
      }
      
      const sites = await siteService.getAvailableSites();
      setAvailableSites(sites);
      
      setIsLoading(false);
    };

    initSite();
  }, []);

  const switchSite = async (siteId: string) => {
    const site = availableSites.find(s => s.id === siteId);
    if (site) {
      // URL frissítés
      if (site.slug !== currentSite?.slug) {
        window.location.href = `https://${site.slug}.formfiller.com`;
      }
    }
  };

  return (
    <SiteContext.Provider value={{ currentSite, availableSites, isLoading, switchSite }}>
      {children}
    </SiteContext.Provider>
  );
}
```

### Hook Használat

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

## API Hívások Site Kontextusban

### Site Header

Minden API hívás tartalmazza az aktuális site azonosítót:

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

### Site-Specifikus Endpoint-ok

```typescript
// services/dataService.ts
export const dataService = {
  async query(configId: string, options: QueryOptions) {
    // Site automatikusan a header-ben
    return apiClient.get(`/api/data/${configId}`, { params: options });
  },

  async save(configId: string, data: any) {
    // Site automatikusan a header-ben
    return apiClient.post(`/api/data/${configId}`, data);
  }
};
```

## Site Váltó Komponens

```typescript
function SiteSwitcher() {
  const { currentSite, availableSites, switchSite } = useSite();
  const { hasPermission } = usePermissions();

  // Csak ha több site-hoz van hozzáférés
  if (availableSites.length <= 1) {
    return null;
  }

  return (
    <DropDownButton
      text={currentSite?.name || 'Site kiválasztása'}
      icon="globe"
      items={availableSites}
      keyExpr="id"
      displayExpr="name"
      onItemClick={e => switchSite(e.itemData.id)}
      itemRender={site => (
        <div className="site-item">
          {site.branding?.logo && <img src={site.branding.logo} alt="" />}
          <span>{site.name}</span>
          {site.id === currentSite?.id && <Icon name="check" />}
        </div>
      )}
    />
  );
}
```

## Site Branding

### Branding Konfiguráció

```typescript
interface SiteBranding {
  logo?: string;
  favicon?: string;
  primaryColor?: string;
  secondaryColor?: string;
  customCss?: string;
}
```

### Branding Alkalmazás

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

      // CSS változók
      if (branding.primaryColor) {
        document.documentElement.style.setProperty('--primary-color', branding.primaryColor);
      }

      // Egyedi CSS
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

## Site-Specifikus Konfigurációk

### Konfiguráció Szűrés

```typescript
function ConfigList() {
  const { currentSite } = useSite();
  const [configs, setConfigs] = useState<Config[]>([]);

  useEffect(() => {
    // API automatikusan szűri site alapján
    const loadConfigs = async () => {
      const data = await configService.getAll();
      setConfigs(data);
    };
    loadConfigs();
  }, [currentSite]); // Site váltáskor újratöltés

  return <DataGrid dataSource={configs} />;
}
```

### Megosztott vs Site-Specifikus Configok

```typescript
interface Config {
  id: string;
  title: string;
  siteId?: string;      // null = globális, minden site-on elérhető
  isGlobal: boolean;    // Kifejezetten megosztott
}

// Szűrés a frontenden (opcionális, backend is szűr)
const visibleConfigs = configs.filter(c => 
  c.isGlobal || c.siteId === currentSite?.id
);
```

## Regisztráció és Site Létrehozás

### Site Létrehozás Regisztrációkor

```typescript
function RegisterWithSite() {
  const [formData, setFormData] = useState({
    // Felhasználói adatok
    name: '',
    email: '',
    password: '',
    // Site adatok
    siteName: '',
    siteSlug: ''
  });

  const [slugAvailable, setSlugAvailable] = useState<boolean | null>(null);

  // Slug elérhetőség ellenőrzés
  const checkSlugAvailability = debounce(async (slug: string) => {
    if (slug.length < 3) return;
    const available = await siteService.checkSlugAvailability(slug);
    setSlugAvailable(available);
  }, 500);

  const handleSubmit = async () => {
    await authService.registerWithSite(formData);
    // Átirányítás az új site-ra
    window.location.href = `https://${formData.siteSlug}.formfiller.com`;
  };

  return (
    <Form>
      {/* User fields */}
      <SimpleItem dataField="name" />
      <SimpleItem dataField="email" />
      <SimpleItem dataField="password" editorType="dxTextBox" editorOptions={{ mode: 'password' }} />
      
      {/* Site fields */}
      <GroupItem caption="Új munkaterület létrehozása">
        <SimpleItem dataField="siteName" label={{ text: 'Munkaterület neve' }} />
        <SimpleItem 
          dataField="siteSlug" 
          label={{ text: 'URL azonosító' }}
          editorOptions={{
            onValueChanged: e => checkSlugAvailability(e.value)
          }}
        />
        {slugAvailable === false && (
          <div className="error">Ez az azonosító már foglalt</div>
        )}
      </GroupItem>
    </Form>
  );
}
```

## Felhasználó Meghívás Site-ra

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
    notify('Meghívó elküldve', 'success');
  };

  return (
    <Form>
      <SimpleItem 
        dataField="email" 
        label={{ text: 'Email cím' }}
        editorOptions={{ value: email, onValueChanged: e => setEmail(e.value) }}
      />
      <SimpleItem
        dataField="role"
        label={{ text: 'Szerepkör' }}
        editorType="dxSelectBox"
        editorOptions={{
          items: availableRoles,
          value: role,
          onValueChanged: e => setRole(e.value)
        }}
      />
      <ButtonItem>
        <ButtonOptions text="Meghívás" onClick={handleInvite} />
      </ButtonItem>
    </Form>
  );
}
```

## Routing Multisite Módban

```typescript
// App.tsx
function App() {
  const { currentSite, isLoading } = useSite();

  if (isLoading) {
    return <LoadingScreen />;
  }

  // Ha nincs site és kötelező
  if (!currentSite && REQUIRE_SITE) {
    return <Navigate to="/select-site" />;
  }

  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/form/:configId" element={<FormPage />} />
      {/* Site admin - csak owner/admin */}
      <Route 
        path="/site-settings" 
        element={
          <PermissionGate role={['owner', 'admin']}>
            <SiteSettingsPage />
          </PermissionGate>
        } 
      />
    </Routes>
  );
}
```

## Best Practices

### 1. Site Kontextus Mindig Elérhető

```typescript
// Biztosítsd, hogy SiteProvider a komponens fa tetején legyen
<AuthProvider>
  <SiteProvider>
    <PermissionProvider>
      <App />
    </PermissionProvider>
  </SiteProvider>
</AuthProvider>
```

### 2. Site Váltás Kezelése

```typescript
// Site váltáskor töröld a cache-t
const switchSite = async (siteId: string) => {
  // Cache törlés
  queryClient.clear();
  
  // Navigáció az új site-ra
  // ...
};
```

### 3. Hibakezelés Site Nélkül

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

