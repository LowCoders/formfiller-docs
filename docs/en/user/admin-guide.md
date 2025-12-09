# Administration Guide

This guide is for system administrators.

## Administrator Permissions

Administrators can perform the following operations:

- User management
- Role and permission configuration
- Configuration (form) creation and modification
- System settings modification
- Site management (in multisite mode)

## User Management

![User management](images/admin-users.png)
*The user management interface*

### List Users

1. Open "Administration" menu
2. Select "Users"

### Create New User

1. Click "New user" button
2. Enter details:
   - Name
   - Email address
   - Role(s)
3. System sends invitation email

### Edit User

1. Click user's name
2. Modify details
3. Save

### Deactivate User

1. Open user details
2. Click "Deactivate" button
3. Confirm

Deactivated user cannot log in, but their data is preserved.

### Delete User

**Warning:** This is a permanent action!

1. Open user details
2. Click "Delete" button
3. Confirm

## Roles and Permissions

![Role management](images/admin-roles.png)
*System roles overview*

### Built-in Roles

| Role | Description |
|------|-------------|
| `admin` | Full access to everything |
| `manager` | Manage users and configurations |
| `owner` | Full management of own site |
| `editor` | Edit configurations and data |
| `contributor` | Create and modify data |
| `creator` | Only create data |
| `viewer` | Read-only access |

### Create Custom Role

1. Open "Roles" menu item
2. Click "New role" button
3. Enter:
   - Name
   - Description
   - Permissions
4. Save

### Permission Settings

Each permission can include the following operations:
- Create
- Read
- Update
- Delete

**Resource types:**
- Configurations (configs)
- Data (data)
- Users (users)
- Roles (roles)
- Sites (sites)

### Assign Role to User

1. Open user details
2. In "Roles" section add/remove roles
3. Save

## Configuration Management

### List Configurations

1. Open "Configurations" menu item
2. Filter by type or tags

### Create New Configuration

1. Click "New configuration" button
2. Select type:
   - Form
   - Grid
   - Tree
3. Enter title and description
4. Design form in editor
5. Publish

### Copy Configuration

1. Open configuration to copy
2. Click "Copy" button
3. Enter new name
4. Modify as needed

### Export/Import Configuration

**Export:**
1. Open configuration
2. Click "Export" button
3. Save JSON file

**Import:**
1. Click "Import" button
2. Select JSON file
3. Review and save

## Site Management (Multisite)

### List Sites

1. Open "Sites" menu item
2. See all sites and their status

### Create New Site

1. Click "New site" button
2. Enter:
   - Site name (appears in URL)
   - Display name
   - Owner email
3. System automatically creates the site

### Site Settings

1. Open the site
2. Modify settings:
   - Name
   - Owner
   - Allowed features
   - Custom settings

### Deactivate/Delete Site

**Deactivate:** Site temporarily unavailable, but data preserved.

**Delete:** Permanently deletes site and all its data.

## System Monitoring

### View Logs

1. Open "Logs" menu item
2. Filter by:
   - Time period
   - User
   - Action type
   - Severity

### System Status

On "System Status" page you can see:
- Number of active users
- Number of stored records
- System resource usage
- Last backup time

## Security Settings

### Password Policy

Configurable:
- Minimum length
- Required characters (upper/lowercase, number, special)
- Expiration time
- Previous password prevention

### Session Settings

- Session timeout
- Concurrent logins count
- IP-based restriction

### Two-Factor Authentication

1. Enable 2FA in system settings
2. Users can individually activate

## Backup and Restore

### Automatic Backup

System creates daily automatic backups. Configurable:
- Backup time
- Retention period
- Storage location

### Manual Backup

1. Open "Backup" menu item
2. Click "Create Backup" button
3. Wait for completion
4. Download backup file

### Restore

**Warning:** Restore overwrites current data!

1. Open "Backup" menu item
2. Select backup to restore
3. Click "Restore" button
4. Confirm

## Troubleshooting

### User Cannot Log In

Check:
1. Is user active
2. Has password expired
3. Is there IP restriction
4. Is site active (in multisite mode)

### Permission Problems

1. Check user's roles
2. Check role permissions
3. Check site-specific permissions

### Performance Problems

1. Check system status
2. Check large configurations
3. Check number of queries

