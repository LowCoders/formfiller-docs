# Event Handling

FormFiller uses a declarative event handling system that can be controlled from JSON configuration.

## Basic Concepts

### Event Types

| Event | Description | Parameters |
|-------|-------------|------------|
| `onValueChanged` | When field value changes | `value`, `previousValue` |
| `onFocusIn` | When field receives focus | - |
| `onFocusOut` | When focus is lost | - |
| `onInitialized` | When field is initialized | - |

### Handler Types

| Handler | Description | Parameters |
|---------|-------------|------------|
| `log` | Console logging | `message` |
| `setValue` | Set value | `target`, `value` |
| `calculate` | Execute calculation | `target`, `formula` |
| `validate` | Trigger validation | `fields` |
| `saveForm` | Save form | - |
| `setVisibility` | Set visibility | `target`, `visible` |
| `setDisabled` | Set disabled | `target`, `disabled` |

## Configuration

### Simple Handler

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

### Multiple Handlers

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

### Conditional Handler

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

## Calculations

### Simple Formula

```json
{
  "handler": "calculate",
  "params": {
    "target": "total",
    "formula": "quantity * price"
  }
}
```

### Complex Formula

```json
{
  "handler": "calculate",
  "params": {
    "target": "finalPrice",
    "formula": "(quantity * price) * (1 - discount / 100)"
  }
}
```

### Available Functions

- `sum(field)` - Sum
- `avg(field)` - Average
- `min(field)` - Minimum
- `max(field)` - Maximum
- `count(field)` - Count
- `round(value, decimals)` - Round
- `floor(value)` - Floor
- `ceil(value)` - Ceiling

## Custom Handler Development

### TypeScript

```typescript
import { EventHandlerRegistry } from '../services/EventHandlerRegistry';

// Register handler
EventHandlerRegistry.register('myCustomHandler', (context, params) => {
  const { formManager, fieldName, value } = context;
  const { targetField, customParam } = params;
  
  // Handler logic
  if (value > 100) {
    formManager.setValue(targetField, customParam);
  }
});
```

### Usage in Configuration

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

## Context Object

The handler receives the context object:

```typescript
interface EventContext {
  formManager: FormManager;  // Form Manager reference
  fieldName: string;         // Name of the field that triggered the event
  value: any;               // Current value
  previousValue: any;       // Previous value
  event: string;            // Event type
  config: FieldConfig;      // Field configuration
}
```

## Examples

### Dynamic Sum

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

### Conditional Fields

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

### Trigger Validation

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

