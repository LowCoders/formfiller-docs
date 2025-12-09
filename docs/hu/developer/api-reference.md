# API Referencia

A FormFiller REST API dokumentációja. Interaktív dokumentáció: `http://localhost:3001/api-docs`

## Autentikáció

### POST /auth/login

Bejelentkezés email és jelszóval.

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

Új felhasználó regisztrálása.

**Request:**
```json
{
  "email": "newuser@example.com",
  "password": "password123",
  "name": "New User"
}
```

### GET /auth/me

Bejelentkezett felhasználó lekérése.

**Headers:** `Authorization: Bearer <token>`

### POST /auth/logout

Kijelentkezés (token blacklist-re kerül).

### POST /auth/change-password

Jelszó módosítása.

**Request:**
```json
{
  "currentPassword": "oldPassword",
  "newPassword": "newPassword123"
}
```

## Konfigurációk

### GET /api/configs

Összes konfiguráció listázása.

**Query params:**
- `type` - Szűrés típus szerint (grid, tree, form)
- `tags` - Szűrés címkék szerint

**Response:**
```json
{
  "data": [
    {
      "id": "...",
      "title": "Munkavállalók",
      "type": "grid",
      "tags": ["hr"]
    }
  ],
  "totalCount": 10
}
```

### GET /api/config/:configId

Konfiguráció lekérése ID alapján.

**Response:**
```json
{
  "id": "...",
  "title": "Munkavállalók",
  "type": "form",
  "config": {
    "items": [...]
  }
}
```

### POST /api/configs

Új konfiguráció létrehozása. (Admin)

**Request:**
```json
{
  "title": "Új Űrlap",
  "type": "form",
  "config": {
    "items": [...]
  },
  "tags": ["test"]
}
```

### PUT /api/configs/:configId

Konfiguráció módosítása. (Admin)

### DELETE /api/configs/:configId

Konfiguráció törlése. (Admin)

## Adatok

### GET /api/data/:configId

Adatok lekérése DevExtreme query formátumban.

**Query params:**
- `skip` - Kihagyandó rekordok száma
- `take` - Lekérendő rekordok száma
- `filter` - DevExtreme filter (JSON)
- `sort` - DevExtreme sort (JSON)
- `group` - Csoportosítás (JSON)
- `totalSummary` - Összesítések (JSON)

**Response:**
```json
{
  "data": [...],
  "totalCount": 100,
  "summary": [...]
}
```

### POST /api/data/:configId

Új rekord létrehozása.

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com"
}
```

### PUT /api/data/:configId/:dataId

Rekord módosítása.

### DELETE /api/data/:configId/:dataId

Rekord törlése.

## Felhasználók

### GET /api/users

Felhasználók listázása. (Admin)

### GET /api/users/:userId

Felhasználó lekérése ID alapján.

### PUT /api/users/:userId

Felhasználó módosítása.

## Workflow

### GET /api/workflow

Workflow-k listázása.

### GET /api/workflow/:workflowId

Workflow lekérése.

### POST /api/workflow

Új workflow létrehozása.

### POST /api/workflow/:workflowId/execute

Workflow végrehajtása.

**Request:**
```json
{
  "data": {...},
  "context": {...}
}
```

## Szerepkörök

### GET /api/roles

Szerepkörök listázása.

### POST /api/roles

Új szerepkör létrehozása. (Admin)

### PUT /api/roles/:roleId

Szerepkör módosítása. (Admin)

### DELETE /api/roles/:roleId

Szerepkör törlése. (Admin)

## Site-ok (Multisite)

### GET /api/sites

Site-ok listázása.

### POST /api/sites

Új site létrehozása.

### GET /api/sites/:siteId

Site lekérése.

### PUT /api/sites/:siteId

Site módosítása.

## Hibakódok

| Kód | Leírás |
|-----|--------|
| 400 | Bad Request - Hibás kérés |
| 401 | Unauthorized - Nincs bejelentkezve |
| 403 | Forbidden - Nincs jogosultság |
| 404 | Not Found - Nem található |
| 409 | Conflict - Ütközés (pl. duplikált email) |
| 422 | Unprocessable Entity - Validációs hiba |
| 500 | Internal Server Error - Szerver hiba |

## Hibaválasz Formátum

```json
{
  "error": {
    "message": "Hibaüzenet",
    "code": "ERROR_CODE",
    "details": {...}
  }
}
```

## Rate Limiting

Az API rate limiting-et alkalmaz:
- 100 request/perc/IP (általános)
- 10 request/perc/IP (autentikáció)

Limit elérése esetén: `429 Too Many Requests`

## CORS

A backend CORS-t támogat. Engedélyezett origin-ök a `.env` fájlban konfigurálhatók.

