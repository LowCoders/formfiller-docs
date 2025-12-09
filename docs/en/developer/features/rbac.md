# Access Control (RBAC)

FormFiller uses Role-Based Access Control (RBAC) for managing user permissions.

## Built-in Roles

| Role | Description | Typical Use |
|------|-------------|-------------|
| `admin` | Full access | System administrator |
| `manager` | Manage users and configs | Manager |
| `owner` | Full management of own site | Site owner |
| `editor` | Edit configs and data | Editor |
| `contributor` | Create and modify data | Contributor |
| `creator` | Only create data | Data entry |
| `viewer` | Read only | Viewer |

## Frontend Permission Management

### PermissionContext

```typescript
import { usePermissions } from '../contexts/PermissionContext';

function MyComponent() {
  const { 
    hasPermission,    // Single permission check
    hasRole,          // Role check
    hasAnyPermission, // Any permission
    hasAllPermissions // All permissions
  } = usePermissions();

  // Usage
  if (hasPermission('configs:write')) {
    return <EditButton />;
  }

  if (hasRole('admin')) {
    return <AdminPanel />;
  }
}
```

### PermissionGate Component

Declarative permission checking:

```typescript
import { PermissionGate } from '../components/permissions/PermissionGate';

function ConfigList() {
  return (
    <div>
      {/* Everyone sees */}
      <ConfigGrid />

      {/* Only with configs:write permission */}
      <PermissionGate permission="configs:write">
        <Button text="New configuration" onClick={handleCreate} />
      </PermissionGate>

      {/* Only with admin role */}
      <PermissionGate role="admin">
        <Button text="System settings" onClick={handleSettings} />
      </PermissionGate>

      {/* Fallback content */}
      <PermissionGate 
        permission="data:delete" 
        fallback={<span>No delete permission</span>}
      >
        <DeleteButton />
      </PermissionGate>
    </div>
  );
}
```

### Conditional Rendering with Hooks

```typescript
function DataGrid() {
  const { hasPermission } = usePermissions();
  
  const columns = [
    { dataField: 'name', caption: 'Name' },
    { dataField: 'status', caption: 'Status' },
    // Only appears with appropriate permission
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

## Menu and Navigation Filtering

### Permission-Based Menu

```typescript
// config/menuConfig.ts
export const menuItems = [
  {
    text: 'Home',
    path: '/',
    icon: 'home'
    // Everyone can see
  },
  {
    text: 'Configurations',
    path: '/configs',
    icon: 'settings',
    permission: 'configs:read'  // Only with this permission
  },
  {
    text: 'Users',
    path: '/admin/users',
    icon: 'people',
    role: 'admin'  // Only with admin role
  }
];
```

### Menu Rendering

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

## Form-Level Permissions

### Field Visibility and Editability

Permission-based behavior can be defined in schema:

```json
{
  "name": "salary",
  "title": "Salary",
  "type": "number",
  "visibleForRoles": ["admin", "hr"],
  "editableForRoles": ["admin"]
}
```

### Frontend Processing

```typescript
function processFieldPermissions(field: FieldConfig, userRoles: string[]) {
  // Visibility check
  if (field.visibleForRoles) {
    const canView = field.visibleForRoles.some(role => userRoles.includes(role));
    if (!canView) {
      return null; // Field not rendered
    }
  }

  // Editability check
  if (field.editableForRoles) {
    const canEdit = field.editableForRoles.some(role => userRoles.includes(role));
    if (!canEdit) {
      return { ...field, editorOptions: { ...field.editorOptions, readOnly: true } };
    }
  }

  return field;
}
```

## Site-Specific Permissions

In multisite mode, permissions can differ per site:

```typescript
interface UserSiteRole {
  siteId: string;
  siteName: string;
  roles: string[];
  permissions: string[];
}

// Current site permissions
const { currentSite, siteRoles } = usePermissions();
const currentSiteRoles = siteRoles.find(sr => sr.siteId === currentSite?.id);
```

## Best Practices

### 1. Always Check on Backend Too

Frontend permission checking only serves UX purposes. Real security is on the backend:

```typescript
// Frontend - UX
if (!hasPermission('data:delete')) {
  return <DisabledButton />;
}

// Backend - Security (always checks)
// DELETE /api/data/:id
// → 403 if no permission
```

### 2. Use Granular Permissions

```typescript
// ❌ Too general
hasRole('admin')

// ✅ Specific
hasPermission('configs:create')
hasPermission('data:export')
```

### 3. Cache Permissions

```typescript
// In PermissionContext
const [permissions, setPermissions] = useState<string[]>([]);

// Only load once unless changed
useEffect(() => {
  if (user && permissions.length === 0) {
    loadPermissions();
  }
}, [user]);
```

### 4. Handle Permission Changes

```typescript
// If admin modifies user permissions
websocket.on('permissions:updated', () => {
  permissionService.refresh();
  notify('Your permissions have been updated', 'info');
});
```

