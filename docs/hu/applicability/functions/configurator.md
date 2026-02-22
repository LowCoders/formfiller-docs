[← Vissza a Funkciók oldalra](index.md)

# Konfigurátor Rendszerek

A konfigurátor (CPQ - Configure, Price, Quote) funkciók területén a FormFiller alapokat biztosít, de teljes körű megoldáshoz jelentős bővítés szükséges.

## Értékelés

| Metrika | Érték |
|---------|-------|
| **Jelenlegi megfelelőség** | ★★★☆☆ |
| **Fejlesztési potenciál** | ★★★★★ |
| **Piaci méret** | $20-40 Mrd |
| **Megtakarítás** | 50-70% |
| **Siker** | Közepes |

## Piaci Megoldások

| Szoftver | Típus | Ár |
|----------|-------|-----|
| Salesforce CPQ | Enterprise | $75-150/felh./hó |
| Configure One | Manufacturing | $50K+/év |
| Tacton | Complex products | $100K+/év |
| Epicor CPQ | Manufacturing | $30K+/év |

## FormFiller Alkalmazási Területek

### Támogatott Funkciók

| Funkció | Támogatás | Megjegyzés |
|---------|:---------:|------------|
| Termékkonfiguráció | ★★★★☆ | Feltételes mezők |
| Szolgáltatás összeállítás | ★★★★☆ | Checkbox, select |
| Függőségi szabályok | ★★★★★ | visibleIf, disabledIf |
| Árkalkuláció (egyszerű) | ★★★☆☆ | ComputedRules |
| Ajánlat generálás | ★★★☆☆ | PDF export bővítéssel |

### Nem Támogatott (natívan)

- Vizuális termék előnézet
- Komplex árszabályok (tiered pricing)
- BOM (Bill of Materials) generálás
- 3D konfigurátor
- ERP integráció

## Bővítési Lehetőségek

| Komponens | Konfigurátor Alkalmazás | Előny |
|-----------|------------------------|-------|
| **TreeView** | Termék hierarchia | Kategória navigáció |
| **Form** | Konfiguráció űrlap | Komplex függőségek |
| **DataGrid** | Termék lista | Szűrés, választás |
| **Charts** | Ár összehasonlítás | Vizuális kalkuláció |
| **Diagram** | Konfiguráció vizualizáció | Összetevők ábrázolása |

## AI Integráció

| AI Funkció | Leírás | Előny |
|------------|--------|-------|
| **Intelligens ajánlás** | Gyakori kombinációk | Upsell/cross-sell |
| **Ár optimalizálás** | Piaci adatok alapján | Versenyképes árak |
| **Kompatibilitás ellenőrzés** | Automatikus validáció | Kevesebb hiba |
| **NL konfiguráció** | "Szeretnék egy basic csomagot API-val" | Egyszerű használat |
| **Prediktív árazás** | Kereslet előrejelzés | Dinamikus ár |

## Pro/Kontra

**Előnyök:**
- Költséghatékony alap konfigurátor
- Komplex függőségi logika
- Gyors testreszabás
- API integráció lehetősége

**Hátrányok:**
- Nincs vizuális termék előnézet
- Komplex árszabályok limitáltak
- Nincs BOM kezelés
- ERP integráció egyedi

## Mikor Válaszd?

| Szcenárió | Ajánlott? |
|-----------|:---------:|
| Szolgáltatás konfigurátor | ✅ Igen |
| Egyszerű termék opciók | ✅ Igen |
| Ajánlatkérés űrlap | ✅ Igen |
| Manufacturing CPQ | ❌ Nem |
| 3D vizualizáció | ❌ Nem |

## Példa: Szolgáltatás Konfigurátor

```json
{
  "name": "serviceConfigurator",
  "title": "Szolgáltatás Konfigurátor",
  "items": [
    {
      "name": "basePackage",
      "type": "radioGroup",
      "label": "Alapcsomag",
      "items": [
        { "value": "starter", "label": "Starter", "price": 9900 },
        { "value": "business", "label": "Business", "price": 29900 },
        { "value": "enterprise", "label": "Enterprise", "price": 99900 }
      ],
      "validationRules": [{ "type": "required" }]
    },
    {
      "name": "users",
      "type": "number",
      "label": "Felhasználók száma",
      "min": 1,
      "max": 1000,
      "defaultValue": 5
    },
    {
      "name": "addons",
      "type": "checkboxGroup",
      "label": "Kiegészítők",
      "items": [
        { "value": "api", "label": "API hozzáférés (+5000 Ft/hó)", "price": 5000 },
        { "value": "sso", "label": "SSO integráció (+10000 Ft/hó)", "price": 10000 },
        { "value": "sla", "label": "Prémium SLA (+15000 Ft/hó)", "price": 15000 }
      ]
    },
    {
      "name": "storage",
      "type": "select",
      "label": "Tárhely",
      "items": [
        { "value": "10gb", "label": "10 GB (alap)", "price": 0 },
        { "value": "100gb", "label": "100 GB (+2000 Ft/hó)", "price": 2000 },
        { "value": "1tb", "label": "1 TB (+10000 Ft/hó)", "price": 10000 }
      ],
      "visibleIf": "basePackage !== 'enterprise'"
    }
  ],
  "computedFields": [
    {
      "name": "basePrice",
      "expression": "getPackagePrice(basePackage)"
    },
    {
      "name": "userPrice",
      "expression": "basePackage === 'starter' ? users * 1000 : users * 500"
    },
    {
      "name": "addonsPrice",
      "expression": "sumAddonPrices(addons)"
    },
    {
      "name": "storagePrice",
      "expression": "getStoragePrice(storage)"
    },
    {
      "name": "monthlyTotal",
      "expression": "basePrice + userPrice + addonsPrice + storagePrice",
      "label": "Havi díj összesen"
    },
    {
      "name": "yearlyTotal",
      "expression": "monthlyTotal * 12 * 0.9",
      "label": "Éves díj (10% kedvezménnyel)"
    }
  ]
}
```

## Bővítési Lehetőségek

| Fejlesztés | Komplexitás |
|------------|:-----------:|
| Termékkatalógus modul | Közepes |
| Tiered pricing | Közepes |
| PDF ajánlat generálás | Alacsony |
| ERP csatlakozó | Magas |
| Vizuális preview | Magas |

---

[← Vissza a Funkciók Összefoglalóhoz](./index.md)
