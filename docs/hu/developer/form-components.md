# Form Komponensek

A FormFiller különböző típusú mezőket és struktúrákat támogat JSON konfiguráció alapján.

## Mező Típusok

### Szöveges Mezők

```json
{
  "name": "firstName",
  "title": "Keresztnév",
  "type": "text",
  "validationRules": [
    { "type": "required", "message": "Kötelező mező" }
  ]
}
```

### Számok

```json
{
  "name": "age",
  "title": "Életkor",
  "type": "number",
  "editorOptions": {
    "min": 0,
    "max": 150,
    "showSpinButtons": true
  }
}
```

### Dátum

```json
{
  "name": "birthDate",
  "title": "Születési dátum",
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
  "title": "Elfogadom a feltételeket",
  "type": "boolean"
}
```

### Lookup (Legördülő)

```json
{
  "name": "country",
  "title": "Ország",
  "type": "lookup",
  "lookup": {
    "dataSource": "/api/countries",
    "valueExpr": "id",
    "displayExpr": "name"
  }
}
```

### Függő Lookup

```json
{
  "name": "city",
  "title": "Város",
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

## Struktúra Elemek

### Csoport

```json
{
  "itemType": "group",
  "caption": "Személyes adatok",
  "colCount": 2,
  "items": [
    { "name": "firstName", "type": "text" },
    { "name": "lastName", "type": "text" }
  ]
}
```

### Fülek

```json
{
  "itemType": "tabbed",
  "tabs": [
    {
      "title": "Alapadatok",
      "items": [
        { "name": "name", "type": "text" }
      ]
    },
    {
      "title": "Kapcsolat",
      "items": [
        { "name": "email", "type": "text" }
      ]
    }
  ]
}
```

### Beágyazott Rács

```json
{
  "name": "items",
  "type": "grid",
  "gridConfig": {
    "allowAdding": true,
    "allowDeleting": true,
    "columns": [
      { "dataField": "name", "caption": "Név" },
      { "dataField": "quantity", "caption": "Mennyiség", "dataType": "number" },
      { "dataField": "price", "caption": "Ár", "dataType": "number" }
    ]
  }
}
```

## Feltételes Megjelenítés

### visibleIf

```json
{
  "name": "spouseName",
  "title": "Házastárs neve",
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
  "title": "Kedvezmény",
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
  "title": "Cégnév",
  "type": "text",
  "requiredIf": {
    "customerType": ["business"]
  }
}
```

## Validációs Szabályok

### Required

```json
{
  "type": "required",
  "message": "Kötelező mező"
}
```

### StringLength

```json
{
  "type": "stringLength",
  "min": 2,
  "max": 100,
  "message": "2-100 karakter között kell lennie"
}
```

### Range

```json
{
  "type": "range",
  "min": 1,
  "max": 100,
  "message": "1 és 100 között kell lennie"
}
```

### Email

```json
{
  "type": "email",
  "message": "Érvényes email cím szükséges"
}
```

### Pattern

```json
{
  "type": "pattern",
  "pattern": "^\\+?[0-9]{9,15}$",
  "message": "Érvényes telefonszám formátum"
}
```

## Editor Options

### Text Editor

```json
{
  "editorOptions": {
    "placeholder": "Írja be a nevét",
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
    "placeholder": "Válasszon..."
  }
}
```

## Teljes Példa

```json
{
  "id": "employee-form",
  "title": "Munkavállaló",
  "type": "form",
  "items": [
    {
      "itemType": "group",
      "caption": "Személyes adatok",
      "colCount": 2,
      "items": [
        {
          "name": "firstName",
          "title": "Keresztnév",
          "type": "text",
          "validationRules": [
            { "type": "required" },
            { "type": "stringLength", "min": 2, "max": 50 }
          ]
        },
        {
          "name": "lastName",
          "title": "Vezetéknév",
          "type": "text",
          "validationRules": [{ "type": "required" }]
        },
        {
          "name": "birthDate",
          "title": "Születési dátum",
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
      "caption": "Munkahely",
      "items": [
        {
          "name": "department",
          "title": "Részleg",
          "type": "lookup",
          "lookup": {
            "dataSource": "/api/departments",
            "valueExpr": "id",
            "displayExpr": "name"
          }
        },
        {
          "name": "position",
          "title": "Pozíció",
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
          "title": "Fizetés",
          "type": "number",
          "editorOptions": {
            "format": "#,##0 Ft",
            "min": 0
          }
        }
      ]
    }
  ]
}
```

