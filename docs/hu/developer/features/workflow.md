# Workflow Kezelés

## Áttekintés

A FormFiller workflow rendszere lehetővé teszi összetett üzleti folyamatok deklaratív definiálását JSON formátumban. A workflow-k lépésekből (steps) állnak, amelyek sorrendben vagy feltételesen hajtódnak végre.

## Workflow Koncepció

### Mi a Workflow?

A workflow egy automatizált folyamat, amely több lépésből áll. Ahelyett, hogy egyedi kódot írnánk minden üzleti folyamathoz, deklaratívan definiáljuk a lépéseket:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Workflow Definíció                               │
│                                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐        │
│  │  Step 1  │────→│  Step 2  │────→│  Step 3  │────→│  Step N  │        │
│  │ validate │     │   save   │     │  notify  │     │   api    │        │
│  └──────────┘     └──────────┘     └──────────┘     └──────────┘        │
│       │                │                │                │               │
│       └────────────────┴────────────────┴────────────────┘               │
│                              │                                           │
│                        Shared Context                                    │
│                  { input, results, savedData }                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### Mire Használható?

| Felhasználási Eset | Példa |
|-------------------|-------|
| **Űrlap feldolgozás** | Validálás → Mentés → Email értesítés |
| **Rendelés kezelés** | Validálás → Készlet ellenőrzés → Fizetés → Mentés → Visszaigazolás |
| **Regisztráció** | Validálás → Felhasználó létrehozás → Üdvözlő email |
| **Jóváhagyási folyamat** | Beküldés → Manager értesítés → Státusz frissítés |
| **Adatexport** | Lekérdezés → Transzformáció → Külső API hívás |

## Workflow Struktúra

### Alap Séma

```json
{
  "name": "order-processing",
  "alias": "order-process",
  "description": "Rendelés feldolgozási workflow",
  "type": "save",
  "config": {
    "steps": [...],
    "errorHandling": {
      "strategy": "stop",
      "notify": true
    },
    "retry": {
      "enabled": true,
      "maxAttempts": 3,
      "backoff": "exponential"
    }
  },
  "permissions": {
    "roles": ["user", "admin"],
    "users": []
  },
  "isActive": true,
  "tags": ["e-commerce", "order"]
}
```

### Workflow Típusok

| Típus | Leírás |
|-------|--------|
| `save` | Adat mentési folyamat |
| `validate` | Csak validációs folyamat |
| `export` | Adat exportálás |
| `import` | Adat importálás |
| `notification` | Értesítési folyamat |
| `custom` | Egyedi folyamat |

## Step Típusok

### 1. Validate Step

Űrlap adatok validálása a konfiguráció schema alapján.

```json
{
  "name": "validate-order",
  "type": "validate",
  "config": {
    "configId": "{{input.configId}}",
    "mode": "sequential",
    "locale": "hu"
  },
  "onError": "stop"
}
```

**Működés:**
1. Betölti a configId által meghatározott űrlap konfigurációt
2. Futtatja a `formfiller-validator` csomagot
3. Végrehajtja a validációs szabályokat és computedRules-t
4. Sikertelen validáció esetén leállítja a workflow-t

### 2. Save Step

Adatok mentése az adatbázisba.

```json
{
  "name": "save-order",
  "type": "save",
  "config": {
    "configId": "{{input.configId}}",
    "locale": "hu"
  },
  "onError": "stop",
  "condition": "validationResult.valid"
}
```

**Működés:**
1. Létrehozza a Data dokumentumot
2. Összekapcsolja a configId-vel és userId-vel
3. Tárolja a computedResults-t (ha engedélyezve)
4. Visszaadja a mentett adat ID-ját

### 3. Notify Step

Email értesítés küldése.

```json
{
  "name": "send-confirmation",
  "type": "notify",
  "config": {
    "email": {
      "to": "admin@example.com",
      "subject": "Új rendelés: {{formTitle}}",
      "format": "html-table"
    }
  },
  "onError": "continue"
}
```

**Email formátumok:**
- `html-table` - Táblázatos HTML formátum
- `plain` - Egyszerű szöveg
- `template` - Egyedi sablon

**Template változók:**
- `{{formTitle}}` - Űrlap címe
- `{{workflowName}}` - Workflow neve
- `{{savedData.*}}` - Mentett adat mezők

### 4. API Step

Külső HTTP API hívás.

```json
{
  "name": "sync-erp",
  "type": "api",
  "config": {
    "url": "https://erp.example.com/api/orders",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer {{env.ERP_TOKEN}}",
      "Content-Type": "application/json"
    },
    "payload": "{{savedData}}"
  },
  "onError": "continue"
}
```

**Támogatott metódusok:** GET, POST, PUT, PATCH, DELETE

**Válasz kezelés:**
```json
{
  "results": {
    "sync-erp": {
      "status": 200,
      "data": { "erpOrderId": "ERP-12345" }
    }
  }
}
```

### 5. Transform Step

Adatok átalakítása a lépések között.

```json
{
  "name": "transform-for-erp",
  "type": "transform",
  "config": {
    "mapping": {
      "customerName": "name",
      "customerEmail": "email",
      "orderItems": "items",
      "totalAmount": "total"
    }
  },
  "onError": "stop"
}
```

**Mapping példa:**
```
Input:                          Output:
{                               {
  "name": "Kiss János",    →      "customerName": "Kiss János",
  "email": "kiss@x.hu",    →      "customerEmail": "kiss@x.hu",
  "items": [...],          →      "orderItems": [...],
  "total": 15000           →      "totalAmount": 15000
}                               }
```

## Hibakezelés

### Hibakezelési Stratégiák

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         onError Stratégiák                               │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ "stop" - Azonnali leállás                                           │ │
│  │                                                                      │ │
│  │  Step 1 ─→ Step 2 ─→ Step 3 ❌ ─X─ Step 4                           │ │
│  │                        ↓                                             │ │
│  │                   Workflow FAIL                                      │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ "continue" - Folytatás a következő lépéssel                         │ │
│  │                                                                      │ │
│  │  Step 1 ─→ Step 2 ─→ Step 3 ⚠️ ─→ Step 4 ─→ SUCCESS                 │ │
│  │                        ↓                                             │ │
│  │                   (logged error)                                     │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ "rollback" - Visszavonás (limitált támogatás)                       │ │
│  │                                                                      │ │
│  │  Step 1 ←─ Step 2 ←─ Step 3 ❌                                       │ │
│  │    ↓                                                                 │ │
│  │  ROLLBACK                                                            │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### Workflow Szintű Hibakezelés

```json
{
  "config": {
    "errorHandling": {
      "strategy": "stop",
      "notify": true
    }
  }
}
```

- `notify: true` - Admin értesítés küldése hiba esetén

## Feltételes Végrehajtás

### Condition Kifejezések

A `condition` mező lehetővé teszi lépések feltételes végrehajtását:

```json
{
  "name": "send-premium-email",
  "type": "notify",
  "condition": "savedData.customerType.equals.premium",
  "config": {
    "email": {
      "to": "{{savedData.email}}",
      "subject": "VIP Visszaigazolás"
    }
  }
}
```

### Elérhető Kontextus Változók

| Változó | Leírás |
|---------|--------|
| `input` | Eredeti bemeneti adatok |
| `savedData` | Mentett adat (save step után) |
| `validationResult` | Validáció eredménye |
| `results.{stepName}` | Adott lépés eredménye |

### Condition Szintaxis

```javascript
// Egyszerű létezés ellenőrzés
"condition": "savedData.exists"

// Nested property
"condition": "validationResult.valid"

// Step eredmény ellenőrzés
"condition": "results.sync-erp.status"
```

## Retry Mechanizmus

### Konfiguráció

```json
{
  "config": {
    "retry": {
      "enabled": true,
      "maxAttempts": 3,
      "backoff": "exponential"
    }
  }
}
```

### Backoff Stratégiák

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Backoff Típusok                                  │
│                                                                          │
│  "fixed" - Fix időköz                                                    │
│  ├─ Attempt 1: 0ms                                                       │
│  ├─ Attempt 2: 1000ms                                                    │
│  └─ Attempt 3: 1000ms                                                    │
│                                                                          │
│  "linear" - Lineárisan növekvő                                           │
│  ├─ Attempt 1: 0ms                                                       │
│  ├─ Attempt 2: 1000ms                                                    │
│  └─ Attempt 3: 2000ms                                                    │
│                                                                          │
│  "exponential" - Exponenciálisan növekvő (ajánlott)                      │
│  ├─ Attempt 1: 0ms                                                       │
│  ├─ Attempt 2: 1000ms                                                    │
│  └─ Attempt 3: 4000ms                                                    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Gyakorlati Példák

### E-commerce Rendelés Feldolgozás

```json
{
  "name": "ecommerce-order-workflow",
  "alias": "order-process",
  "description": "Teljes rendelés feldolgozási folyamat",
  "type": "save",
  "config": {
    "steps": [
      {
        "name": "validate-order",
        "type": "validate",
        "config": {
          "mode": "sequential",
          "locale": "hu"
        },
        "onError": "stop"
      },
      {
        "name": "transform-order",
        "type": "transform",
        "config": {
          "mapping": {
            "orderId": "orderNumber",
            "customer": "customerData",
            "items": "orderItems",
            "shipping": "deliveryAddress"
          }
        },
        "onError": "stop"
      },
      {
        "name": "save-order",
        "type": "save",
        "config": {
          "locale": "hu"
        },
        "onError": "stop",
        "condition": "validationResult.valid"
      },
      {
        "name": "sync-inventory",
        "type": "api",
        "config": {
          "url": "https://inventory.example.com/api/reserve",
          "method": "POST",
          "headers": {
            "Authorization": "Bearer {{env.INVENTORY_TOKEN}}"
          },
          "payload": {
            "orderId": "{{savedData._id}}",
            "items": "{{savedData.data.items}}"
          }
        },
        "onError": "continue"
      },
      {
        "name": "notify-customer",
        "type": "notify",
        "config": {
          "email": {
            "to": "{{input.email}}",
            "subject": "Rendelés visszaigazolás - {{formTitle}}",
            "format": "html-table"
          }
        },
        "onError": "continue"
      },
      {
        "name": "notify-warehouse",
        "type": "notify",
        "config": {
          "email": {
            "to": "warehouse@example.com",
            "subject": "Új rendelés: {{savedData._id}}",
            "format": "html-table"
          }
        },
        "onError": "continue"
      }
    ],
    "errorHandling": {
      "strategy": "stop",
      "notify": true
    },
    "retry": {
      "enabled": true,
      "maxAttempts": 3,
      "backoff": "exponential"
    }
  },
  "permissions": {
    "roles": ["user", "admin"],
    "users": []
  },
  "isActive": true,
  "tags": ["e-commerce", "order", "notification"]
}
```

### Jóváhagyási Folyamat

```json
{
  "name": "approval-workflow",
  "alias": "approval",
  "description": "Dokumentum jóváhagyási folyamat",
  "type": "custom",
  "config": {
    "steps": [
      {
        "name": "validate-document",
        "type": "validate",
        "config": {
          "mode": "sequential"
        },
        "onError": "stop"
      },
      {
        "name": "save-pending",
        "type": "save",
        "config": {
          "locale": "hu"
        },
        "onError": "stop"
      },
      {
        "name": "notify-approver",
        "type": "notify",
        "config": {
          "email": {
            "to": "{{input.approverEmail}}",
            "subject": "Jóváhagyás szükséges: {{formTitle}}",
            "format": "html-table"
          }
        },
        "onError": "continue"
      },
      {
        "name": "create-task",
        "type": "api",
        "config": {
          "url": "https://tasks.example.com/api/tasks",
          "method": "POST",
          "payload": {
            "type": "approval",
            "documentId": "{{savedData._id}}",
            "assignee": "{{input.approverId}}",
            "dueDate": "{{input.dueDate}}"
          }
        },
        "onError": "continue"
      }
    ],
    "errorHandling": {
      "strategy": "continue",
      "notify": true
    }
  },
  "permissions": {
    "roles": ["user", "manager"],
    "users": []
  }
}
```

### Vizsga Kiértékelés

```json
{
  "name": "exam-evaluation",
  "alias": "exam-eval",
  "description": "Vizsga automatikus kiértékelése és eredmény küldése",
  "type": "save",
  "config": {
    "steps": [
      {
        "name": "validate-answers",
        "type": "validate",
        "config": {
          "mode": "sequential",
          "locale": "hu"
        },
        "onError": "stop"
      },
      {
        "name": "save-results",
        "type": "save",
        "config": {
          "locale": "hu"
        },
        "onError": "stop"
      },
      {
        "name": "notify-student",
        "type": "notify",
        "config": {
          "email": {
            "to": "{{input.studentEmail}}",
            "subject": "Vizsga eredmény - {{formTitle}}",
            "format": "html-table"
          }
        },
        "onError": "continue"
      },
      {
        "name": "notify-instructor",
        "type": "notify",
        "config": {
          "email": {
            "to": "instructor@example.com",
            "subject": "Vizsga beadva: {{input.studentName}}",
            "format": "plain"
          }
        },
        "onError": "continue",
        "condition": "computedResults.totalScore"
      }
    ],
    "errorHandling": {
      "strategy": "stop",
      "notify": true
    }
  }
}
```

## API Használat

### Workflow Létrehozás

```bash
POST /api/workflow
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "my-workflow",
  "type": "save",
  "config": {
    "steps": [...]
  }
}
```

### Workflow Végrehajtás

```bash
POST /api/workflow/:workflowId/execute
Content-Type: application/json
Authorization: Bearer <token>

{
  "configId": "form-config-id",
  "data": {
    "field1": "value1",
    "field2": "value2"
  }
}
```

### Válasz Struktúra

```json
{
  "success": true,
  "message": "Workflow executed successfully",
  "workflowId": "workflow-id",
  "workflowName": "my-workflow",
  "steps": [
    {
      "name": "validate-order",
      "type": "validate",
      "status": "success",
      "result": { "valid": true }
    },
    {
      "name": "save-order",
      "type": "save",
      "status": "success",
      "result": { "dataId": "data-id" }
    }
  ],
  "data": {
    "_id": "saved-data-id",
    "configId": "form-config-id",
    "data": {...}
  },
  "results": {
    "validate-order": { "valid": true },
    "save-order": { "dataId": "data-id" }
  }
}
```

## Best Practices

### 1. Lépések Elnevezése

```json
// ✅ Jó - Leíró, akció-alapú nevek
"name": "validate-customer-data"
"name": "save-order-to-database"
"name": "notify-warehouse-team"

// ❌ Rossz - Általános nevek
"name": "step1"
"name": "validate"
"name": "email"
```

### 2. Hibakezelés Tervezése

```json
// Kritikus lépések: stop
{
  "name": "save-payment",
  "type": "save",
  "onError": "stop"  // Fizetés nem sikerülhet csendben
}

// Opcionális lépések: continue
{
  "name": "send-analytics",
  "type": "api",
  "onError": "continue"  // Analytics hiba ne állítsa le
}
```

### 3. Condition Használat

```json
// Csak sikeres mentés után értesít
{
  "name": "notify-user",
  "type": "notify",
  "condition": "savedData.exists"
}

// Csak prémium ügyfeleknek
{
  "name": "premium-notification",
  "type": "notify",
  "condition": "savedData.data.membershipLevel.equals.premium"
}
```

### 4. Retry API Hívásokhoz

```json
{
  "name": "external-api-call",
  "type": "api",
  "config": {
    "url": "https://external.api.com/endpoint"
  },
  "onError": "continue"
}

// Workflow szinten engedélyezett retry
"retry": {
  "enabled": true,
  "maxAttempts": 3,
  "backoff": "exponential"
}
```

## Kapcsolódó Dokumentációk

- [Backend Fejlesztés](../backend.md) - Backend architektúra
- [Adatkezelés](./data-management.md) - DataService és mentés
- [Validáció](../validation.md) - Validációs szabályok
- [Schema](../schema.md) - Űrlap konfiguráció

