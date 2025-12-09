# Validation

FormFiller uses two-tier validation: frontend validation for immediate feedback, and backend validation for security.

## Frontend Validation

DevExtreme-compatible validation rules from JSON configuration.

### Basic Validations

```json
{
  "name": "email",
  "validationRules": [
    { "type": "required", "message": "Email is required" },
    { "type": "email", "message": "Valid email address required" }
  ]
}
```

### All Validation Types

| Type | Description | Parameters |
|------|-------------|------------|
| `required` | Required field | - |
| `email` | Email format | - |
| `numeric` | Numbers only | - |
| `stringLength` | String length | `min`, `max` |
| `range` | Number range | `min`, `max` |
| `pattern` | Regex pattern | `pattern` |
| `compare` | Field comparison | `comparisonTarget`, `comparisonType` |
| `custom` | Custom validator | `validationCallback` |

### StringLength Examples

```json
// Minimum length
{ "type": "stringLength", "min": 2, "message": "At least 2 characters" }

// Maximum length
{ "type": "stringLength", "max": 100, "message": "Maximum 100 characters" }

// Min and max
{ "type": "stringLength", "min": 8, "max": 50, "message": "Between 8-50 characters" }
```

### Range Examples

```json
// Minimum value
{ "type": "range", "min": 0, "message": "Cannot be negative" }

// Maximum value
{ "type": "range", "max": 100, "message": "Maximum 100" }

// Min and max
{ "type": "range", "min": 1, "max": 100, "message": "Between 1 and 100" }
```

### Pattern Examples

```json
// Phone number
{ 
  "type": "pattern", 
  "pattern": "^\\+?[0-9]{9,15}$", 
  "message": "Valid phone number" 
}

// Postal code
{ 
  "type": "pattern", 
  "pattern": "^[0-9]{5}$", 
  "message": "5-digit postal code" 
}
```

### Compare

```json
// Password confirmation
{
  "name": "passwordConfirm",
  "validationRules": [
    { 
      "type": "compare", 
      "comparisonTarget": "password",
      "comparisonType": "==",
      "message": "Passwords must match" 
    }
  ]
}
```

## Conditional Validation

### requiredIf

Conditionally required field:

```json
{
  "name": "companyTaxNumber",
  "requiredIf": {
    "customerType": ["business"]
  },
  "validationRules": [
    { "type": "pattern", "pattern": "^[0-9]{8}-[0-9]-[0-9]{2}$" }
  ]
}
```

### Dynamic Validation

With event handler:

```json
{
  "name": "quantity",
  "onValueChanged": [
    {
      "handler": "validate",
      "params": { "fields": ["total"] }
    }
  ]
}
```

## Validation Levels

Validation can be defined at two levels:

### Field Level Validation

Validation rules attached directly to the field:

```json
{
  "name": "email",
  "validationRules": [
    { "type": "required" },
    { "type": "email" }
  ]
}
```

### Form Level Validation

Global rules defined in the form configuration's `validationRules` field:

```json
{
  "type": "form",
  "title": "Registration",
  "validationRules": [
    {
      "type": "crossField",
      "fields": ["password", "passwordConfirm"],
      "rule": "equals",
      "message": "Passwords must match"
    }
  ],
  "items": [...]
}
```

The advantage of form-level validation is that rules affecting multiple fields can be managed in one place.

## Group Validators

Group validators validate logically related fields together. Particularly useful for complex business rules.

### Group Validator Definition

```json
{
  "type": "form",
  "validationGroups": [
    {
      "name": "addressValidation",
      "fields": ["country", "postalCode", "city", "street"],
      "rules": [
        {
          "type": "custom",
          "message": "US addresses require 5-digit postal code",
          "condition": "country === 'US' && !/^[0-9]{5}$/.test(postalCode)"
        }
      ]
    },
    {
      "name": "dateRangeValidation", 
      "fields": ["startDate", "endDate"],
      "rules": [
        {
          "type": "custom",
          "message": "Start date cannot be later than end date",
          "condition": "startDate > endDate"
        }
      ]
    }
  ]
}
```

### Logical Operators in Group Validation

```json
{
  "validationGroups": [
    {
      "name": "contactValidation",
      "operator": "OR",
      "rules": [
        { "field": "email", "type": "required" },
        { "field": "phone", "type": "required" }
      ],
      "message": "At least one contact method is required (email or phone)"
    }
  ]
}
```

Supported operators:
- `AND` - All rules must be met (default)
- `OR` - At least one rule must be met

## Field References with Full Path

For nested structures (groups, tabs, nested grids), field references use **full paths**. This approach is not just a technical solution, but brings **significant advantages** in terms of data structuring and management.

### Advantages of Path-Based Approach

| Advantage | Description |
|-----------|-------------|
| **Complex data structure** | Data can be stored in natural hierarchy |
| **Semantic meaning** | The path itself carries meaning (`billing.address` vs `shipping.address`) |
| **Form-data correspondence** | Saved data reflects the form's logical structure |
| **Easy navigation** | Unambiguous reference to nested fields |
| **MongoDB optimal** | Embedded documents, no JOINs |

### Path Syntax

```
simple field:       fieldName
group field:        groupName.fieldName
deep nesting:       level1.level2.level3.fieldName
tabbed:             tabbedName.tabName.fieldName
grid row:           gridName[rowIndex].columnName
all grid rows:      gridName[*].columnName
```

### Data Structure Example

A form definition:
```json
{
  "items": [
    {
      "type": "group",
      "name": "personalData",
      "items": [
        { "name": "firstName", "type": "text" },
        { "name": "lastName", "type": "text" }
      ]
    },
    {
      "type": "group",
      "name": "billingAddress",
      "items": [
        { "name": "street", "type": "text" },
        { "name": "city", "type": "text" },
        { "name": "postalCode", "type": "text" }
      ]
    }
  ]
}
```

The saved data reflects the path structure:
```json
{
  "personalData": {
    "firstName": "John",
    "lastName": "Smith"
  },
  "billingAddress": {
    "street": "123 Main St",
    "city": "New York",
    "postalCode": "10001"
  }
}
```

This means:
- The **form's logical structure** is reflected in the data
- The **context** of fields is preserved (`street` is part of `billingAddress`)
- Database queries are **intuitive**: `{ "billingAddress.city": "New York" }`

### Examples

#### Reference to Group Field

```json
{
  "validationGroups": [
    {
      "name": "billingCheck",
      "fields": [
        "personalData.firstName",
        "personalData.lastName",
        "billingAddress.street",
        "billingAddress.city"
      ],
      "rules": [...]
    }
  ]
}
```

#### Reference to Nested Grid Field

```json
{
  "name": "totalAmount",
  "validationRules": [
    {
      "type": "custom",
      "dependsOn": ["orderItems[*].quantity", "orderItems[*].unitPrice"],
      "message": "Amount doesn't match the sum of items"
    }
  ]
}
```

The `[*]` syntax refers to all rows in the grid.

#### Conditional Validation with Path

```json
{
  "name": "billingAddress.postalCode",
  "requiredIf": {
    "shippingOptions.differentBillingAddress": [true]
  }
}
```

## ComputedRules (Computed Rules)

`computedRules` is not a classic validator - it doesn't throw errors, but **calculates values** based on other fields. Typical use: exam sheet evaluation, score calculation, grade determination.

### Basic Concepts

| Property | Description |
|----------|-------------|
| `computedRules` | Array of calculation rules |
| `targetField` | Target field where result goes |
| `formula` | Calculation formula |
| `conditions` | Conditional value assignment |

### Simple Score Calculation

```json
{
  "type": "form",
  "computedRules": [
    {
      "targetField": "totalScore",
      "formula": "math1 + math2 + math3 + literature + history"
    },
    {
      "targetField": "average",
      "formula": "totalScore / 5"
    }
  ]
}
```

### Exam Sheet Evaluation with Conditions

```json
{
  "computedRules": [
    {
      "targetField": "examResult",
      "conditions": [
        { "when": "totalScore >= 90", "value": "Excellent (A)" },
        { "when": "totalScore >= 75", "value": "Good (B)" },
        { "when": "totalScore >= 60", "value": "Average (C)" },
        { "when": "totalScore >= 40", "value": "Pass (D)" },
        { "when": "totalScore < 40", "value": "Fail (F)" }
      ]
    }
  ]
}
```

### Complex Evaluation Logic

```json
{
  "computedRules": [
    {
      "targetField": "practicalGrade",
      "conditions": [
        { 
          "when": "practicalScore >= 80 && attendance >= 90", 
          "value": "Passed - Excellent" 
        },
        { 
          "when": "practicalScore >= 60 && attendance >= 75", 
          "value": "Passed" 
        },
        { 
          "when": "practicalScore < 60 || attendance < 75", 
          "value": "Failed" 
        }
      ]
    },
    {
      "targetField": "canTakeExam",
      "formula": "practicalGrade !== 'Failed'"
    }
  ]
}
```

### Weighted Average Calculation

```json
{
  "computedRules": [
    {
      "targetField": "weightedAverage",
      "formula": "(math * 2 + physics * 2 + literature * 1 + history * 1) / 6",
      "description": "Math and physics with double weight"
    }
  ]
}
```

### Nested Grid Summation

```json
{
  "computedRules": [
    {
      "targetField": "orderTotal",
      "formula": "sum(orderItems[*].lineTotal)",
      "description": "Sum of all items"
    },
    {
      "targetField": "orderItems[*].lineTotal",
      "formula": "orderItems[$index].quantity * orderItems[$index].unitPrice",
      "scope": "row",
      "description": "Line item total"
    }
  ]
}
```

`$index` is the current row index in the grid.

### Available Functions

| Function | Description | Example |
|----------|-------------|---------|
| `sum(field)` | Sum | `sum(items[*].price)` |
| `avg(field)` | Average | `avg(scores[*].value)` |
| `min(field)` | Minimum | `min(bids[*].amount)` |
| `max(field)` | Maximum | `max(scores[*].value)` |
| `count(field)` | Count | `count(items[*])` |
| `round(value, decimals)` | Round | `round(average, 2)` |
| `floor(value)` | Floor | `floor(price)` |
| `ceil(value)` | Ceiling | `ceil(price)` |
| `if(condition, then, else)` | Conditional value | `if(score >= 50, 'Pass', 'Fail')` |

### ComputedRules vs ValidationRules

| Property | ValidationRules | ComputedRules |
|----------|-----------------|---------------|
| Purpose | Signal errors | Calculate values |
| Output | valid/invalid + error message | Calculated value |
| When runs | Before save | On field change |
| Blocks save | Yes | No |
| Typical use | Required fields, formats | Totals, averages, grades |

## Backend Validation

The `formfiller-validator` package provides advanced validation.

### Basic Usage

```typescript
import { Validator } from 'formfiller-validator';

const validator = new Validator({
  mode: 'parallel',
  cache: { enabled: true }
});

const result = await validator.validate(formData, config);

if (!result.valid) {
  return res.status(422).json({ 
    errors: result.errors 
  });
}
```

### Validation Modes

```typescript
// Parallel - faster for large forms
const validator = new Validator({ mode: 'parallel' });

// Sequential - easier debugging
const validator = new Validator({ mode: 'sequential' });
```

### Caching

```typescript
const validator = new Validator({
  cache: {
    enabled: true,
    ttl: 3600 // in seconds
  }
});
```

### Custom Validator

```typescript
import { ValidationRule } from 'formfiller-validator';

const customRule: ValidationRule = {
  type: 'custom',
  validate: async (value, context) => {
    // Async validation (e.g. DB check)
    const exists = await checkIfExists(value);
    if (exists) {
      return { valid: false, message: 'Already exists' };
    }
    return { valid: true };
  }
};
```

### Cross-Field Validation

```typescript
const crossFieldRule = {
  type: 'crossField',
  fields: ['startDate', 'endDate'],
  validate: (values) => {
    if (values.startDate > values.endDate) {
      return { 
        valid: false, 
        message: 'Start date cannot be later' 
      };
    }
    return { valid: true };
  }
};
```

## Validation Result

```typescript
interface ValidationResult {
  valid: boolean;
  errors: ValidationError[];
}

interface ValidationError {
  field: string;       // Field name
  message: string;     // Error message
  type: string;        // Validation type
  value?: any;         // Invalid value
}
```

## Customizing Error Messages

### Static Message

```json
{
  "type": "required",
  "message": "This field is required"
}
```

### Dynamic Message

```json
{
  "type": "stringLength",
  "min": 8,
  "message": "At least {{min}} characters required"
}
```

### Localized Message

```json
{
  "type": "required",
  "message": "validation:required"
}
```

Translation in the `locales/` directory:
```json
{
  "validation": {
    "required": "This field is required"
  }
}
```

