# Jogosultságkezelés (RBAC)

A FormFiller szerepkör alapú hozzáférés-vezérlést (RBAC - Role-Based Access Control) használ a felhasználói jogosultságok kezelésére.

## RBAC Architektúra

```mermaid
flowchart TB
    U["👤 FELHASZNÁLÓ"]
    
    subgraph Roles["SZEREPKÖRÖK"]
        R1[admin]
        R2[manager]
        R3[editor]
        R4[contributor]
        R5[viewer]
    end
    
    subgraph Permissions["ENGEDÉLYEK"]
        P1["configs:read"]
        P2["configs:write"]
        P3["data:read"]
        P4["data:write"]
        P5["users:manage"]
        P6["roles:manage"]
        P7["sites:manage"]
    end
    
    subgraph Resources["ERŐFORRÁSOK"]
        RES1[Konfigurációk]
        RES2[Adatok]
        RES3[Felhasználók]
        RES4[Szerepkörök]
    end
    
    U --> Roles
    Roles --> Permissions
    Permissions --> Resources
```

## Beépített Szerepkörök

| Szerepkör | Leírás | Tipikus Használat |
|-----------|--------|-------------------|
| `admin` | Teljes hozzáférés | Rendszergazda |
| `manager` | Felhasználók és konfigok kezelése | Vezető |
| `owner` | Saját site teljes kezelése | Site tulajdonos |
| `editor` | Konfigok és adatok szerkesztése | Szerkesztő |
| `contributor` | Adatok létrehozása és módosítása | Munkatárs |
| `creator` | Csak adatok létrehozása | Adatbevitel |
| `viewer` | Csak olvasás | Megtekintő |

## Frontend Jogosultság Kezelés

### PermissionContext

```typescript
import { usePermissions } from '../contexts/PermissionContext';

function MyComponent() {
  const { 
    hasPermission,    // Egyedi engedély ellenőrzés
    hasRole,          // Szerepkör ellenőrzés
    hasAnyPermission, // Bármelyik engedély
    hasAllPermissions // Összes engedély
  } = usePermissions();

  // Használat
  if (hasPermission('configs:write')) {
    return <EditButton />;
  }

  if (hasRole('admin')) {
    return <AdminPanel />;
  }
}
```

### PermissionGate Komponens

Deklaratív jogosultság ellenőrzés:

```typescript
import { PermissionGate } from '../components/permissions/PermissionGate';

function ConfigList() {
  return (
    <div>
      {/* Mindenki látja */}
      <ConfigGrid />

      {/* Csak configs:write engedéllyel */}
      <PermissionGate permission="configs:write">
        <Button text="Új konfiguráció" onClick={handleCreate} />
      </PermissionGate>

      {/* Csak admin szerepkörrel */}
      <PermissionGate role="admin">
        <Button text="Rendszer beállítások" onClick={handleSettings} />
      </PermissionGate>

      {/* Fallback tartalom */}
      <PermissionGate 
        permission="data:delete" 
        fallback={<span>Nincs törlési jogod</span>}
      >
        <DeleteButton />
      </PermissionGate>
    </div>
  );
}
```

### Feltételes Renderelés Hook-kal

```typescript
function DataGrid() {
  const { hasPermission } = usePermissions();
  
  const columns = [
    { dataField: 'name', caption: 'Név' },
    { dataField: 'status', caption: 'Státusz' },
    // Csak megfelelő jogosultsággal jelenik meg
    ...(hasPermission('data:write') ? [
      { 
        type: 'buttons',
        buttons: ['edit', 'delete']
      }
    ] : [])
  ];

  return <DataGrid columns={columns} />;
}
```

## Menü és Navigáció Szűrés

### Jogosultság Alapú Menü

```typescript
// config/menuConfig.ts
export const menuItems = [
  {
    text: 'Kezdőlap',
    path: '/',
    icon: 'home'
    // Mindenki láthatja
  },
  {
    text: 'Konfigurációk',
    path: '/configs',
    icon: 'settings',
    permission: 'configs:read'  // Csak ezzel az engedéllyel
  },
  {
    text: 'Felhasználók',
    path: '/admin/users',
    icon: 'people',
    role: 'admin'  // Csak admin szerepkörrel
  },
  {
    text: 'Szerepkörök',
    path: '/admin/roles',
    icon: 'security',
    permissions: ['roles:read', 'roles:manage']  // Bármelyik elég
  }
];
```

### Menü Renderelés

```typescript
function Navigation() {
  const { hasPermission, hasRole } = usePermissions();

  const filteredMenu = menuItems.filter(item => {
    if (item.permission && !hasPermission(item.permission)) return false;
    if (item.role && !hasRole(item.role)) return false;
    if (item.permissions && !item.permissions.some(p => hasPermission(p))) return false;
    return true;
  });

  return (
    <nav>
      {filteredMenu.map(item => (
        <NavLink key={item.path} to={item.path}>
          <Icon name={item.icon} />
          {item.text}
        </NavLink>
      ))}
    </nav>
  );
}
```

## Form Szintű Jogosultságok

### Mező Láthatóság és Szerkeszthetőség

A schema-ban definiálható jogosultság-alapú viselkedés:

```json
{
  "name": "salary",
  "title": "Fizetés",
  "type": "number",
  "visibleForRoles": ["admin", "hr"],
  "editableForRoles": ["admin"]
}
```

### Frontend Feldolgozás

```typescript
function processFieldPermissions(field: FieldConfig, userRoles: string[]) {
  // Láthatóság ellenőrzés
  if (field.visibleForRoles) {
    const canView = field.visibleForRoles.some(role => userRoles.includes(role));
    if (!canView) {
      return null; // Mező nem renderelődik
    }
  }

  // Szerkeszthetőség ellenőrzés
  if (field.editableForRoles) {
    const canEdit = field.editableForRoles.some(role => userRoles.includes(role));
    if (!canEdit) {
      return { ...field, editorOptions: { ...field.editorOptions, readOnly: true } };
    }
  }

  return field;
}
```

## Site-Specifikus Jogosultságok

Multisite módban a jogosultságok site-onként eltérhetnek:

```typescript
interface UserSiteRole {
  siteId: string;
  siteName: string;
  roles: string[];
  permissions: string[];
}

// Aktuális site jogosultságai
const { currentSite, siteRoles } = usePermissions();
const currentSiteRoles = siteRoles.find(sr => sr.siteId === currentSite?.id);
```

### Site Váltás

```typescript
function SiteSwitcher() {
  const { user, switchSite, currentSite } = useAuth();

  return (
    <SelectBox
      items={user.sites}
      valueExpr="id"
      displayExpr="name"
      value={currentSite?.id}
      onValueChanged={e => switchSite(e.value)}
    />
  );
}
```

## API Hívások és Jogosultságok

### Jogosultság Ellenőrzés API Előtt

```typescript
async function deleteConfig(configId: string) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission('configs:delete')) {
    notify('Nincs törlési jogosultságod', 'error');
    return;
  }

  try {
    await configService.delete(configId);
    notify('Törölve', 'success');
  } catch (error) {
    if (error.response?.status === 403) {
      notify('Hozzáférés megtagadva', 'error');
    }
  }
}
```

### 403 Hibakezelés

```typescript
// api/client.ts interceptor
apiClient.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 403) {
      notify('Nincs jogosultságod ehhez a művelethez', 'error');
      // Opcionálisan: jogosultságok újratöltése
      permissionService.refresh();
    }
    return Promise.reject(error);
  }
);
```

## Admin Felület

### Szerepkör Kezelés

```typescript
function RoleManagement() {
  return (
    <PermissionGate role="admin" fallback={<AccessDenied />}>
      <DataGrid
        dataSource={rolesDataSource}
        columns={[
          { dataField: 'name', caption: 'Szerepkör neve' },
          { dataField: 'displayName', caption: 'Megjelenített név' },
          { dataField: 'permissions', caption: 'Engedélyek', cellRender: PermissionTags }
        ]}
        editing={{ allowAdding: true, allowUpdating: true, allowDeleting: true }}
      />
    </PermissionGate>
  );
}
```

### Felhasználó Szerepkör Hozzárendelés

```typescript
function UserRoleAssignment({ userId }: { userId: string }) {
  const [userRoles, setUserRoles] = useState<string[]>([]);
  const [availableRoles, setAvailableRoles] = useState<Role[]>([]);

  const handleRoleChange = async (roleId: string, assigned: boolean) => {
    if (assigned) {
      await userService.assignRole(userId, roleId);
    } else {
      await userService.removeRole(userId, roleId);
    }
    refreshUserRoles();
  };

  return (
    <List
      items={availableRoles}
      itemRender={role => (
        <CheckBox
          text={role.displayName}
          value={userRoles.includes(role.id)}
          onValueChanged={e => handleRoleChange(role.id, e.value)}
        />
      )}
    />
  );
}
```

## Best Practices

### 1. Mindig Ellenőrizd a Backend-en Is

A frontend jogosultság ellenőrzés csak UX célokat szolgál. A valódi biztonság a backend-en van:

```typescript
// Frontend - UX
if (!hasPermission('data:delete')) {
  return <DisabledButton />;
}

// Backend - Biztonság (mindig ellenőriz)
// DELETE /api/data/:id
// → 403 ha nincs jogosultság
```

### 2. Használj Granulált Engedélyeket

```typescript
// ❌ Túl általános
hasRole('admin')

// ✅ Specifikus
hasPermission('configs:create')
hasPermission('data:export')
```

### 3. Cache-elj Jogosultságokat

```typescript
// PermissionContext-ben
const [permissions, setPermissions] = useState<string[]>([]);

// Csak egyszer töltjük be, hacsak nem változik
useEffect(() => {
  if (user && permissions.length === 0) {
    loadPermissions();
  }
}, [user]);
```

### 4. Kezeld a Jogosultság Változásokat

```typescript
// Ha admin módosítja a felhasználó jogait
websocket.on('permissions:updated', () => {
  permissionService.refresh();
  notify('Jogosultságaid frissültek', 'info');
});
```

