# API Reference

FormFiller REST API documentation. Interactive documentation: `http://localhost:3001/api-docs`

## Authentication

### POST /auth/login

Login with email and password.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "...",
    "email": "user@example.com",
    "name": "User Name"
  }
}
```

### POST /auth/register

Register new user.

**Request:**
```json
{
  "email": "newuser@example.com",
  "password": "password123",
  "name": "New User"
}
```

### GET /auth/me

Get logged in user.

**Headers:** `Authorization: Bearer <token>`

### POST /auth/logout

Logout (token gets blacklisted).

### POST /auth/change-password

Change password.

**Request:**
```json
{
  "currentPassword": "oldPassword",
  "newPassword": "newPassword123"
}
```

## Configurations

### GET /api/configs

List all configurations.

**Query params:**
- `type` - Filter by type (grid, tree, form)
- `tags` - Filter by tags

**Response:**
```json
{
  "data": [
    {
      "id": "...",
      "title": "Employees",
      "type": "grid",
      "tags": ["hr"]
    }
  ],
  "totalCount": 10
}
```

### GET /api/config/:configId

Get configuration by ID.

**Response:**
```json
{
  "id": "...",
  "title": "Employees",
  "type": "form",
  "config": {
    "items": [...]
  }
}
```

### POST /api/configs

Create new configuration. (Admin)

**Request:**
```json
{
  "title": "New Form",
  "type": "form",
  "config": {
    "items": [...]
  },
  "tags": ["test"]
}
```

### PUT /api/configs/:configId

Update configuration. (Admin)

### DELETE /api/configs/:configId

Delete configuration. (Admin)

## Data

### GET /api/data/:configId

Get data in DevExtreme query format.

**Query params:**
- `skip` - Number of records to skip
- `take` - Number of records to retrieve
- `filter` - DevExtreme filter (JSON)
- `sort` - DevExtreme sort (JSON)
- `group` - Grouping (JSON)
- `totalSummary` - Summaries (JSON)

**Response:**
```json
{
  "data": [...],
  "totalCount": 100,
  "summary": [...]
}
```

### POST /api/data/:configId

Create new record.

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com"
}
```

### PUT /api/data/:configId/:dataId

Update record.

### DELETE /api/data/:configId/:dataId

Delete record.

## Users

### GET /api/users

List users. (Admin)

### GET /api/users/:userId

Get user by ID.

### PUT /api/users/:userId

Update user.

## Workflow

### GET /api/workflow

List workflows.

### GET /api/workflow/:workflowId

Get workflow.

### POST /api/workflow

Create new workflow.

### POST /api/workflow/:workflowId/execute

Execute workflow.

**Request:**
```json
{
  "data": {...},
  "context": {...}
}
```

## Roles

### GET /api/roles

List roles.

### POST /api/roles

Create new role. (Admin)

### PUT /api/roles/:roleId

Update role. (Admin)

### DELETE /api/roles/:roleId

Delete role. (Admin)

## Sites (Multisite)

### GET /api/sites

List sites.

### POST /api/sites

Create new site.

### GET /api/sites/:siteId

Get site.

### PUT /api/sites/:siteId

Update site.

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Bad Request - Invalid request |
| 401 | Unauthorized - Not logged in |
| 403 | Forbidden - No permission |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Conflict (e.g. duplicate email) |
| 422 | Unprocessable Entity - Validation error |
| 500 | Internal Server Error - Server error |

## Error Response Format

```json
{
  "error": {
    "message": "Error message",
    "code": "ERROR_CODE",
    "details": {...}
  }
}
```

## Rate Limiting

The API applies rate limiting:
- 100 requests/minute/IP (general)
- 10 requests/minute/IP (authentication)

When limit is reached: `429 Too Many Requests`

## CORS

The backend supports CORS. Allowed origins can be configured in the `.env` file.

