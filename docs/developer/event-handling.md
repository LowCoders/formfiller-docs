# Eseménykezelés

A FormFiller deklaratív eseménykezelő rendszert használ, amely JSON konfigurációból vezérelhető.

## Alapfogalmak

### Esemény Típusok

| Esemény | Leírás | Paraméterek |
|---------|--------|-------------|
| `onValueChanged` | Mező érték változásakor | `value`, `previousValue` |
| `onFocusIn` | Mező fókuszba kerülésekor | - |
| `onFocusOut` | Fókusz elvesztésekor | - |
| `onInitialized` | Mező inicializálásakor | - |

### Handler Típusok

| Handler | Leírás | Paraméterek |
|---------|--------|-------------|
| `log` | Konzol logolás | `message` |
| `setValue` | Érték beállítása | `target`, `value` |
| `calculate` | Számítás végrehajtása | `target`, `formula` |
| `validate` | Validáció triggerelése | `fields` |
| `saveForm` | Űrlap mentése | - |
| `setVisibility` | Láthatóság beállítása | `target`, `visible` |
| `setDisabled` | Letiltás beállítása | `target`, `disabled` |

## Konfiguráció

### Egyszerű Handler

```json
{
  "name": "email",
  "type": "text",
  "onValueChanged": [
    {
      "handler": "log",
      "params": {
        "message": "Email changed to: {{value}}"
      }
    }
  ]
}
```

### Több Handler

```json
{
  "name": "quantity",
  "type": "number",
  "onValueChanged": [
    {
      "handler": "log",
      "params": { "message": "Quantity: {{value}}" }
    },
    {
      "handler": "calculate",
      "params": {
        "target": "total",
        "formula": "quantity * unitPrice"
      }
    },
    {
      "handler": "validate",
      "params": { "fields": ["total"] }
    }
  ]
}
```

### Feltételes Handler

```json
{
  "name": "paymentMethod",
  "type": "select",
  "onValueChanged": [
    {
      "handler": "setVisibility",
      "condition": "value === 'card'",
      "params": {
        "target": "cardNumber",
        "visible": true
      }
    }
  ]
}
```

## Számítások

### Egyszerű Képlet

```json
{
  "handler": "calculate",
  "params": {
    "target": "total",
    "formula": "quantity * price"
  }
}
```

### Összetett Képlet

```json
{
  "handler": "calculate",
  "params": {
    "target": "finalPrice",
    "formula": "(quantity * price) * (1 - discount / 100)"
  }
}
```

### Elérhető Függvények

- `sum(field)` - Összegzés
- `avg(field)` - Átlag
- `min(field)` - Minimum
- `max(field)` - Maximum
- `count(field)` - Darabszám
- `round(value, decimals)` - Kerekítés
- `floor(value)` - Lefele kerekítés
- `ceil(value)` - Felfele kerekítés

## Egyéni Handler Fejlesztése

### TypeScript

```typescript
import { EventHandlerRegistry } from '../services/EventHandlerRegistry';

// Handler regisztrálása
EventHandlerRegistry.register('myCustomHandler', (context, params) => {
  const { formManager, fieldName, value } = context;
  const { targetField, customParam } = params;
  
  // Handler logika
  if (value > 100) {
    formManager.setValue(targetField, customParam);
  }
});
```

### Használat Konfigurációban

```json
{
  "name": "amount",
  "type": "number",
  "onValueChanged": [
    {
      "handler": "myCustomHandler",
      "params": {
        "targetField": "status",
        "customParam": "high"
      }
    }
  ]
}
```

## Context Objektum

A handler megkapja a context objektumot:

```typescript
interface EventContext {
  formManager: FormManager;  // Form Manager referencia
  fieldName: string;         // Eseményt kiváltó mező neve
  value: any;               // Aktuális érték
  previousValue: any;       // Előző érték
  event: string;            // Esemény típusa
  config: FieldConfig;      // Mező konfigurációja
}
```

## Példák

### Dinamikus Összegzés

```json
{
  "items": [
    {
      "name": "item1Price",
      "type": "number",
      "onValueChanged": [
        {
          "handler": "calculate",
          "params": { "target": "subtotal", "formula": "item1Price + item2Price + item3Price" }
        }
      ]
    },
    {
      "name": "subtotal",
      "type": "number",
      "editorOptions": { "readOnly": true }
    }
  ]
}
```

### Feltételes Mezők

```json
{
  "items": [
    {
      "name": "hasAddress",
      "type": "boolean",
      "onValueChanged": [
        {
          "handler": "setVisibility",
          "params": { "target": "addressGroup", "visible": "{{value}}" }
        }
      ]
    },
    {
      "itemType": "group",
      "name": "addressGroup",
      "visible": false,
      "items": [
        { "name": "street", "type": "text" },
        { "name": "city", "type": "text" }
      ]
    }
  ]
}
```

### Validáció Triggerelés

```json
{
  "name": "password",
  "type": "text",
  "onValueChanged": [
    {
      "handler": "validate",
      "params": { "fields": ["passwordConfirm"] }
    }
  ]
}
```

