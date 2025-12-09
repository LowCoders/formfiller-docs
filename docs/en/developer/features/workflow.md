# Workflow Management

## Overview

The FormFiller workflow system enables declarative definition of complex business processes in JSON format. Workflows consist of steps that execute sequentially or conditionally.

## What is a Workflow?

A workflow is an automated process consisting of multiple steps. Instead of writing custom code for every business process, we define steps declaratively:

### Use Cases

| Use Case | Example |
|----------|---------|
| **Form processing** | Validation → Save → Email notification |
| **Order management** | Validation → Inventory check → Payment → Save → Confirmation |
| **Registration** | Validation → User creation → Welcome email |
| **Approval process** | Submission → Manager notification → Status update |
| **Data export** | Query → Transformation → External API call |

## Workflow Structure

### Basic Schema

```json
{
  "name": "order-processing",
  "alias": "order-process",
  "description": "Order processing workflow",
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

## Step Types

### 1. Validate Step

Form data validation based on configuration schema.

```json
{
  "name": "validate-order",
  "type": "validate",
  "config": {
    "configId": "{{input.configId}}",
    "mode": "sequential",
    "locale": "en"
  },
  "onError": "stop"
}
```

### 2. Save Step

Save data to database.

```json
{
  "name": "save-order",
  "type": "save",
  "config": {
    "configId": "{{input.configId}}",
    "locale": "en"
  },
  "onError": "stop",
  "condition": "validationResult.valid"
}
```

### 3. Notify Step

Send email notification.

```json
{
  "name": "send-confirmation",
  "type": "notify",
  "config": {
    "email": {
      "to": "admin@example.com",
      "subject": "New order: {{formTitle}}",
      "format": "html-table"
    }
  },
  "onError": "continue"
}
```

### 4. API Step

External HTTP API call.

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

### 5. Transform Step

Transform data between steps.

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

## Error Handling

### Error Handling Strategies

- **stop**: Immediate stop on error
- **continue**: Continue to next step on error
- **rollback**: Rollback on error (limited support)

### Workflow-Level Error Handling

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

- `notify: true` - Send admin notification on error

## Conditional Execution

### Condition Expressions

The `condition` field enables conditional step execution:

```json
{
  "name": "send-premium-email",
  "type": "notify",
  "condition": "savedData.customerType.equals.premium",
  "config": {
    "email": {
      "to": "{{savedData.email}}",
      "subject": "VIP Confirmation"
    }
  }
}
```

### Available Context Variables

| Variable | Description |
|----------|-------------|
| `input` | Original input data |
| `savedData` | Saved data (after save step) |
| `validationResult` | Validation result |
| `results.{stepName}` | Result of given step |

## Retry Mechanism

### Configuration

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

### Backoff Strategies

- **fixed**: Fixed interval
- **linear**: Linearly increasing
- **exponential**: Exponentially increasing (recommended)

## Practical Examples

### E-commerce Order Processing

```json
{
  "name": "ecommerce-order-workflow",
  "type": "save",
  "config": {
    "steps": [
      {
        "name": "validate-order",
        "type": "validate",
        "config": { "mode": "sequential", "locale": "en" },
        "onError": "stop"
      },
      {
        "name": "save-order",
        "type": "save",
        "config": { "locale": "en" },
        "onError": "stop",
        "condition": "validationResult.valid"
      },
      {
        "name": "notify-customer",
        "type": "notify",
        "config": {
          "email": {
            "to": "{{input.email}}",
            "subject": "Order confirmation - {{formTitle}}",
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
            "subject": "New order: {{savedData._id}}",
            "format": "html-table"
          }
        },
        "onError": "continue"
      }
    ],
    "errorHandling": { "strategy": "stop", "notify": true },
    "retry": { "enabled": true, "maxAttempts": 3, "backoff": "exponential" }
  }
}
```

## API Usage

### Workflow Execution

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

### Response Structure

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
  }
}
```

## Best Practices

### 1. Step Naming

```json
// ✅ Good - Descriptive, action-based names
"name": "validate-customer-data"
"name": "save-order-to-database"
"name": "notify-warehouse-team"

// ❌ Bad - Generic names
"name": "step1"
"name": "validate"
"name": "email"
```

### 2. Error Handling Design

```json
// Critical steps: stop
{
  "name": "save-payment",
  "type": "save",
  "onError": "stop"  // Payment must not fail silently
}

// Optional steps: continue
{
  "name": "send-analytics",
  "type": "api",
  "onError": "continue"  // Analytics error shouldn't stop workflow
}
```

## Related Documentation

- [Backend Development](../backend.md) - Backend architecture
- [Data Management](./data-management.md) - DataService and saving
- [Validation](../validation.md) - Validation rules
- [Schema](../schema.md) - Form configuration

