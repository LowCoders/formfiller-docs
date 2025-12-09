# Form Components

FormFiller supports various types of fields and structures based on JSON configuration.

## Field Types

### Text Fields

```json
{
  "name": "firstName",
  "title": "First Name",
  "type": "text",
  "validationRules": [
    { "type": "required", "message": "Required field" }
  ]
}
```

### Numbers

```json
{
  "name": "age",
  "title": "Age",
  "type": "number",
  "editorOptions": {
    "min": 0,
    "max": 150,
    "showSpinButtons": true
  }
}
```

### Date

```json
{
  "name": "birthDate",
  "title": "Birth Date",
  "type": "date",
  "editorOptions": {
    "displayFormat": "yyyy-MM-dd",
    "max": "now"
  }
}
```

### Boolean

```json
{
  "name": "acceptTerms",
  "title": "I accept the terms",
  "type": "boolean"
}
```

### Lookup (Dropdown)

```json
{
  "name": "country",
  "title": "Country",
  "type": "lookup",
  "lookup": {
    "dataSource": "/api/countries",
    "valueExpr": "id",
    "displayExpr": "name"
  }
}
```

### Dependent Lookup

```json
{
  "name": "city",
  "title": "City",
  "type": "lookup",
  "lookup": {
    "dataSource": "/api/cities",
    "valueExpr": "id",
    "displayExpr": "name",
    "dependsOn": ["country"],
    "filterExpr": "countryId = {{country}}"
  }
}
```

## Structure Elements

### Group

```json
{
  "itemType": "group",
  "caption": "Personal Data",
  "colCount": 2,
  "items": [
    { "name": "firstName", "type": "text" },
    { "name": "lastName", "type": "text" }
  ]
}
```

### Tabs

```json
{
  "itemType": "tabbed",
  "tabs": [
    {
      "title": "Basic Data",
      "items": [
        { "name": "name", "type": "text" }
      ]
    },
    {
      "title": "Contact",
      "items": [
        { "name": "email", "type": "text" }
      ]
    }
  ]
}
```

### Nested Grid

```json
{
  "name": "items",
  "type": "grid",
  "gridConfig": {
    "allowAdding": true,
    "allowDeleting": true,
    "columns": [
      { "dataField": "name", "caption": "Name" },
      { "dataField": "quantity", "caption": "Quantity", "dataType": "number" },
      { "dataField": "price", "caption": "Price", "dataType": "number" }
    ]
  }
}
```

## Conditional Display

### visibleIf

```json
{
  "name": "spouseName",
  "title": "Spouse Name",
  "type": "text",
  "visibleIf": {
    "maritalStatus": ["married", "divorced"]
  }
}
```

### disabledIf

```json
{
  "name": "discount",
  "title": "Discount",
  "type": "number",
  "disabledIf": {
    "customerType": ["new"]
  }
}
```

### requiredIf

```json
{
  "name": "companyName",
  "title": "Company Name",
  "type": "text",
  "requiredIf": {
    "customerType": ["business"]
  }
}
```

## Validation Rules

### Required

```json
{
  "type": "required",
  "message": "Required field"
}
```

### StringLength

```json
{
  "type": "stringLength",
  "min": 2,
  "max": 100,
  "message": "Must be between 2-100 characters"
}
```

### Range

```json
{
  "type": "range",
  "min": 1,
  "max": 100,
  "message": "Must be between 1 and 100"
}
```

### Email

```json
{
  "type": "email",
  "message": "Valid email address required"
}
```

### Pattern

```json
{
  "type": "pattern",
  "pattern": "^\\+?[0-9]{9,15}$",
  "message": "Valid phone number format"
}
```

## Editor Options

### Text Editor

```json
{
  "editorOptions": {
    "placeholder": "Enter your name",
    "maxLength": 100,
    "mode": "text"
  }
}
```

### Number Editor

```json
{
  "editorOptions": {
    "min": 0,
    "max": 1000,
    "step": 0.01,
    "showSpinButtons": true,
    "format": "#,##0.00"
  }
}
```

### Date Editor

```json
{
  "editorOptions": {
    "displayFormat": "yyyy-MM-dd",
    "type": "date",
    "min": "2000-01-01",
    "max": "now"
  }
}
```

### SelectBox

```json
{
  "editorOptions": {
    "searchEnabled": true,
    "searchExpr": "name",
    "placeholder": "Select..."
  }
}
```

## Complete Example

```json
{
  "id": "employee-form",
  "title": "Employee",
  "type": "form",
  "items": [
    {
      "itemType": "group",
      "caption": "Personal Data",
      "colCount": 2,
      "items": [
        {
          "name": "firstName",
          "title": "First Name",
          "type": "text",
          "validationRules": [
            { "type": "required" },
            { "type": "stringLength", "min": 2, "max": 50 }
          ]
        },
        {
          "name": "lastName",
          "title": "Last Name",
          "type": "text",
          "validationRules": [{ "type": "required" }]
        },
        {
          "name": "birthDate",
          "title": "Birth Date",
          "type": "date",
          "editorOptions": { "max": "now" }
        },
        {
          "name": "email",
          "title": "Email",
          "type": "text",
          "validationRules": [
            { "type": "required" },
            { "type": "email" }
          ]
        }
      ]
    },
    {
      "itemType": "group",
      "caption": "Workplace",
      "items": [
        {
          "name": "department",
          "title": "Department",
          "type": "lookup",
          "lookup": {
            "dataSource": "/api/departments",
            "valueExpr": "id",
            "displayExpr": "name"
          }
        },
        {
          "name": "position",
          "title": "Position",
          "type": "lookup",
          "lookup": {
            "dataSource": "/api/positions",
            "valueExpr": "id",
            "displayExpr": "title",
            "dependsOn": ["department"],
            "filterExpr": "departmentId = {{department}}"
          }
        },
        {
          "name": "salary",
          "title": "Salary",
          "type": "number",
          "editorOptions": {
            "format": "$#,##0",
            "min": 0
          }
        }
      ]
    }
  ]
}
```

