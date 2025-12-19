# Validáció

A FormFiller kétszintű validációt használ: frontend validáció azonnali visszajelzéshez, és backend validáció a biztonságért.

## Frontend Validáció

DevExtreme-kompatibilis validációs szabályok JSON konfigurációból.

### Alap Validációk

```json
{
  "name": "email",
  "validationRules": [
    { "type": "required", "message": "Az email kötelező" },
    { "type": "email", "message": "Érvényes email cím szükséges" }
  ]
}
```

### Összes Validáció Típus

| Típus | Leírás | Paraméterek |
|-------|--------|-------------|
| `required` | Kötelező mező | - |
| `email` | Email formátum | - |
| `numeric` | Csak számok | - |
| `stringLength` | Szöveghossz | `min`, `max` |
| `range` | Számtartomány | `min`, `max` |
| `pattern` | Regex minta | `pattern` |
| `compare` | Mezők összehasonlítása | `comparisonTarget`, `comparisonType` |
| `custom` | Egyéni validátor | `validationCallback` |

### StringLength Példák

```json
// Minimum hossz
{ "type": "stringLength", "min": 2, "message": "Legalább 2 karakter" }

// Maximum hossz
{ "type": "stringLength", "max": 100, "message": "Maximum 100 karakter" }

// Min és max
{ "type": "stringLength", "min": 8, "max": 50, "message": "8-50 karakter között" }
```

### Range Példák

```json
// Minimum érték
{ "type": "range", "min": 0, "message": "Nem lehet negatív" }

// Maximum érték
{ "type": "range", "max": 100, "message": "Maximum 100" }

// Min és max
{ "type": "range", "min": 1, "max": 100, "message": "1 és 100 között" }
```

### Pattern Példák

```json
// Telefonszám
{ 
  "type": "pattern", 
  "pattern": "^\\+?[0-9]{9,15}$", 
  "message": "Érvényes telefonszám" 
}

// Irányítószám
{ 
  "type": "pattern", 
  "pattern": "^[0-9]{4}$", 
  "message": "4 számjegyű irányítószám" 
}
```

### Compare

```json
// Jelszó megerősítés
{
  "name": "passwordConfirm",
  "validationRules": [
    { 
      "type": "compare", 
      "comparisonTarget": "password",
      "comparisonType": "==",
      "message": "A jelszavaknak egyezniük kell" 
    }
  ]
}
```

## Feltételes Validáció

### requiredIf

Feltételesen kötelező mező:

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

### Dinamikus Validáció

Eseménykezelővel:

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

## Validáció Szintjei

A validáció két szinten definiálható:

### Mező Szintű Validáció

A validációs szabályok közvetlenül a mezőhöz rendelhetők:

```json
{
  "name": "email",
  "validationRules": [
    { "type": "required" },
    { "type": "email" }
  ]
}
```

### Form Szintű Validáció

A form konfiguráció `validationRules` mezőjében globális szabályok definiálhatók:

```json
{
  "type": "form",
  "title": "Regisztráció",
  "validationRules": [
    {
      "type": "crossField",
      "fields": ["password", "passwordConfirm"],
      "rule": "equals",
      "message": "A jelszavaknak egyezniük kell"
    }
  ],
  "items": [...]
}
```

A form szintű validáció előnye, hogy több mezőt érintő szabályokat egy helyen lehet kezelni.

## Group Validátorok

A group validátorok logikailag összetartozó mezőket validálnak együtt. Különösen hasznosak összetett üzleti szabályoknál.

### Group Validátor Definíció

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
          "message": "Magyar cím esetén 4 számjegyű irányítószám szükséges",
          "condition": "country === 'HU' && !/^[0-9]{4}$/.test(postalCode)"
        }
      ]
    },
    {
      "name": "dateRangeValidation", 
      "fields": ["startDate", "endDate"],
      "rules": [
        {
          "type": "custom",
          "message": "A kezdő dátum nem lehet későbbi a végdátumnál",
          "condition": "startDate > endDate"
        }
      ]
    }
  ]
}
```

### Logikai Operátorok Group Validációban

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
      "message": "Legalább egy elérhetőség megadása kötelező (email vagy telefon)"
    }
  ]
}
```

Támogatott operátorok:
- `AND` - Minden szabálynak teljesülnie kell (alapértelmezett)
- `OR` - Legalább egy szabálynak teljesülnie kell

## Mezőhivatkozások Teljes Path-tal

Beágyazott struktúrák (csoportok, fülek, beágyazott rácsok) esetén a mezőhivatkozások **teljes path-ot** használnak. Ez a megközelítés nem csupán technikai megoldás, hanem **jelentős előnyökkel** jár az adatok strukturálása és kezelése szempontjából.

### A Path Alapú Megközelítés Előnyei

| Előny | Leírás |
|-------|--------|
| **Komplex adatstruktúra** | Az adatok természetes hierarchiában tárolhatók |
| **Szemantikus jelentés** | A path maga hordoz jelentést (`billing.address` vs `shipping.address`) |
| **Űrlap-adat megfelelés** | A mentett adat tükrözi az űrlap logikai struktúráját |
| **Könnyű navigáció** | Egyértelmű hivatkozás beágyazott mezőkre |
| **MongoDB optimális** | Beágyazott dokumentumok, nincs JOIN |

### Path Szintaxis

```
egyszerű mező:       fieldName
csoport mező:        groupName.fieldName
mély beágyazás:      level1.level2.level3.fieldName
tabulátor:           tabbedName.tabName.fieldName
grid sor:            gridName[rowIndex].columnName
összes grid sor:     gridName[*].columnName
```

### Adatstruktúra Példa

Egy űrlap definíció:
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

A mentett adat a path struktúrát tükrözi:
```json
{
  "personalData": {
    "firstName": "Kiss",
    "lastName": "János"
  },
  "billingAddress": {
    "street": "Fő utca 1.",
    "city": "Budapest",
    "postalCode": "1011"
  }
}
```

Ez azt jelenti, hogy:
- Az **űrlap logikai struktúrája** visszaköszön az adatokban
- A mezők **kontextusa** megmarad (a `street` a `billingAddress` része)
- Az adatbázis lekérdezések **intuitívak**: `{ "billingAddress.city": "Budapest" }`

### Példák

#### Csoport Mezőre Hivatkozás

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

#### Beágyazott Rács Mezőre Hivatkozás

```json
{
  "name": "totalAmount",
  "validationRules": [
    {
      "type": "custom",
      "dependsOn": ["orderItems[*].quantity", "orderItems[*].unitPrice"],
      "message": "Az összeg nem egyezik a tételek összegével"
    }
  ]
}
```

A `[*]` szintaxis a rács összes sorára vonatkozik.

#### Feltételes Validáció Path-tal

```json
{
  "name": "billingAddress.postalCode",
  "requiredIf": {
    "shippingOptions.differentBillingAddress": [true]
  }
}
```

## ComputedRules (Számított Szabályok)

A `computedRules` nem klasszikus validátor - nem dob hibát, hanem **értéket számít** más mezők alapján. Tipikus felhasználás: vizsgalapok értékelése, pontszámítás, osztályzatok meghatározása.

### Alapfogalmak

| Tulajdonság | Leírás |
|-------------|--------|
| `computedRules` | Számítási szabályok tömbje |
| `targetField` | A cél mező, ahova az eredmény kerül |
| `formula` | Számítási képlet |
| `conditions` | Feltételes értékadás |

### Egyszerű Pontszámítás

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

### Vizsgalap Értékelése Feltételekkel

```json
{
  "computedRules": [
    {
      "targetField": "examResult",
      "conditions": [
        { "when": "totalScore >= 90", "value": "Kiváló (5)" },
        { "when": "totalScore >= 75", "value": "Jó (4)" },
        { "when": "totalScore >= 60", "value": "Közepes (3)" },
        { "when": "totalScore >= 40", "value": "Elégséges (2)" },
        { "when": "totalScore < 40", "value": "Elégtelen (1)" }
      ]
    }
  ]
}
```

### Összetett Értékelési Logika

```json
{
  "computedRules": [
    {
      "targetField": "practicalGrade",
      "conditions": [
        { 
          "when": "practicalScore >= 80 && attendance >= 90", 
          "value": "Megfelelt - Kiváló" 
        },
        { 
          "when": "practicalScore >= 60 && attendance >= 75", 
          "value": "Megfelelt" 
        },
        { 
          "when": "practicalScore < 60 || attendance < 75", 
          "value": "Nem felelt meg" 
        }
      ]
    },
    {
      "targetField": "canTakeExam",
      "formula": "practicalGrade !== 'Nem felelt meg'"
    }
  ]
}
```

### Súlyozott Átlagszámítás

```json
{
  "computedRules": [
    {
      "targetField": "weightedAverage",
      "formula": "(math * 2 + physics * 2 + literature * 1 + history * 1) / 6",
      "description": "Matematika és fizika dupla súllyal"
    }
  ]
}
```

### Beágyazott Rács Összegzése

```json
{
  "computedRules": [
    {
      "targetField": "orderTotal",
      "formula": "sum(orderItems[*].lineTotal)",
      "description": "Összes tétel összege"
    },
    {
      "targetField": "orderItems[*].lineTotal",
      "formula": "orderItems[$index].quantity * orderItems[$index].unitPrice",
      "scope": "row",
      "description": "Sortétel összeg"
    }
  ]
}
```

A `$index` a rács aktuális sorának indexe.

### Elérhető Függvények

| Függvény | Leírás | Példa |
|----------|--------|-------|
| `sum(field)` | Összegzés | `sum(items[*].price)` |
| `avg(field)` | Átlag | `avg(scores[*].value)` |
| `min(field)` | Minimum | `min(bids[*].amount)` |
| `max(field)` | Maximum | `max(scores[*].value)` |
| `count(field)` | Darabszám | `count(items[*])` |
| `round(value, decimals)` | Kerekítés | `round(average, 2)` |
| `floor(value)` | Lefele kerekítés | `floor(price)` |
| `ceil(value)` | Felfele kerekítés | `ceil(price)` |
| `if(condition, then, else)` | Feltételes érték | `if(score >= 50, 'Pass', 'Fail')` |

### ComputedRules vs ValidationRules

| Tulajdonság | ValidationRules | ComputedRules |
|-------------|-----------------|---------------|
| Cél | Hibák jelzése | Értékek számítása |
| Kimenet | valid/invalid + hibaüzenet | Számított érték |
| Mikor fut | Mentés előtt | Mező változáskor |
| Blokkolja mentést | Igen | Nem |
| Tipikus használat | Kötelező mezők, formátumok | Összegek, átlagok, osztályzatok |

## Backend Validáció

A `formfiller-validator` csomag fejlett validációt biztosít.

### Alap Használat

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

### Validációs Módok

```typescript
// Párhuzamos - gyorsabb nagy űrlapoknál
const validator = new Validator({ mode: 'parallel' });

// Szekvenciális - egyszerűbb hibakeresés
const validator = new Validator({ mode: 'sequential' });
```

### Gyorsítótárazás

```typescript
const validator = new Validator({
  cache: {
    enabled: true,
    ttl: 3600 // másodpercben
  }
});
```

### Egyéni Validátor

```typescript
import { ValidationRule } from 'formfiller-validator';

const customRule: ValidationRule = {
  type: 'custom',
  validate: async (value, context) => {
    // Aszinkron validáció (pl. DB ellenőrzés)
    const exists = await checkIfExists(value);
    if (exists) {
      return { valid: false, message: 'Már létezik' };
    }
    return { valid: true };
  }
};
```

### Keresztmező Validáció

```typescript
const crossFieldRule = {
  type: 'crossField',
  fields: ['startDate', 'endDate'],
  validate: (values) => {
    if (values.startDate > values.endDate) {
      return { 
        valid: false, 
        message: 'A kezdő dátum nem lehet későbbi' 
      };
    }
    return { valid: true };
  }
};
```

## Validációs Eredmény

```typescript
interface ValidationResult {
  valid: boolean;
  errors: ValidationError[];
}

interface ValidationError {
  field: string;       // Mező neve
  message: string;     // Hibaüzenet
  type: string;        // Validáció típusa
  value?: any;         // Hibás érték
}
```

## Hibaüzenetek Testreszabása

### Statikus Üzenet

```json
{
  "type": "required",
  "message": "Ez a mező kötelező"
}
```

### Dinamikus Üzenet

```json
{
  "type": "stringLength",
  "min": 8,
  "message": "Legalább {{min}} karakter szükséges"
}
```

### Lokalizált Üzenet

```json
{
  "type": "required",
  "message": "validation:required"
}
```

A fordítás a `locales/` könyvtárban:
```json
{
  "validation": {
    "required": "Ez a mező kötelező"
  }
}
```

## CrossField Validáció (Új Formátum)

A crossField validátorok több mező együttes validálását teszik lehetővé. Az új formátumban a típus közvetlenül jelzi a validátor típusát.

### Elérhető CrossField Típusok

| Típus | Leírás |
|-------|--------|
| `crossFieldEquals` | Mezők egyezőségét ellenőrzi |
| `crossFieldNotEquals` | Mezők nem egyezőségét ellenőrzi |
| `crossFieldGreaterThan` | Első mező > második mező |
| `crossFieldLessThan` | Első mező < második mező |
| `crossFieldSumEquals` | Mezők összege egyenlő egy értékkel |
| `crossFieldPercentageSum` | Mezők összege 100% |
| `crossFieldDateInRange` | Dátum más dátumok tartományában |
| `crossFieldAtLeastOne` | Legalább egy mező kitöltve |
| `crossFieldCustom` | Egyéni validátor |

### Új Formátum Példa

```json
{
  "name": "total_vacation_days",
  "validationRules": [
    {
      "type": "crossFieldSumEquals",
      "targetFields": ["base_vacation_days", "extra_vacation_days"],
      "message": "Az összes szabadságnak egyeznie kell az alap + extra értékkel"
    }
  ]
}
```

### Régi vs Új Formátum

**Régi (továbbra is támogatott):**
```json
{
  "type": "crossField",
  "targetFields": ["password", "passwordConfirm"],
  "crossFieldValidator": "passwordMatch",
  "message": "A jelszavaknak egyezniük kell"
}
```

**Új (ajánlott):**
```json
{
  "type": "crossFieldEquals",
  "targetFields": ["password", "passwordConfirm"],
  "message": "A jelszavaknak egyezniük kell"
}
```

### Automatikus Mező Hozzáadás

Ha egy crossField validátort mező szinten definiálsz, az aktuális mező automatikusan bekerül a `targetFields` tömbbe:

```json
{
  "name": "passwordConfirm",
  "validationRules": [
    {
      "type": "crossFieldEquals",
      "targetFields": ["password"],
      "message": "A jelszavaknak egyezniük kell"
    }
  ]
}
```

A `passwordConfirm` automatikusan bekerül a `targetFields` közé validáláskor.

## Hibamegjelenítés Testreszabása

### errorTarget Property

Az `errorTarget` property segítségével megadhatod, hogy hol jelenjen meg a validációs hiba:

| Érték | Leírás |
|-------|--------|
| `'currentField'` | Csak az aktuális mezőnél (mező szintű alapértelmezés) |
| `'allTargetFields'` | Minden érintett mezőnél (form szintű alapértelmezés) |
| `string[]` | Konkrét mező path-ok, ahol a hiba megjelenjen |

```json
{
  "type": "crossFieldEquals",
  "targetFields": ["field1", "field2", "field3"],
  "message": "A mezőknek egyezniük kell",
  "errorTarget": ["field1", "field3"]
}
```

### Form Szintű Hibapanel

A `FormPreferences`-ben beállítható, hogy a validációs hibák megjelenjenek-e az űrlap tetején (accordion formában):

```json
{
  "preferences": {
    "showErrorsOnFormLevel": true
  }
}
```

| Property | Típus | Alapérték | Leírás |
|----------|-------|-----------|--------|
| `showErrorsOnFormLevel` | `boolean` | `true` | Hibapanel megjelenítése az űrlap tetején |

A form szintű hibák mindig accordion formában jelennek meg. Ha a hibát közvetlenül a mezőnél szeretnéd megjeleníteni, használd az `errorTarget` property-t a validációs szabályban.

### Hierarchikus Címkék

A form szintű hibamegjelenítésnél a mezők hierarchikus címkékkel jelennek meg:

```
⚠️ Validációs Hibák (3)

• Személyes adatok > Lakcím > Irányítószám: Kötelező mező
• Összeg Validáció > Összes szabadság: Az összegnek egyeznie kell
• Kapcsolat > Email: Érvénytelen email formátum
```

A címkék a mező path-ja alapján épülnek fel, a `label`, `caption` vagy `name` property-k felhasználásával.

