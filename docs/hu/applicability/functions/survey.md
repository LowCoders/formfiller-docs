[← Vissza a Funkciók oldalra](index.md)

# Felmérések / Kérdőívek

A felmérések és kérdőívek területe a FormFiller egyik legerősebb alkalmazási területe, ahol natív támogatással és komplex validációval kiváló eredményeket érhet el.

## Értékelés

| Metrika | Érték |
|---------|-------|
| **Jelenlegi megfelelőség** | ★★★★★ |
| **Fejlesztési potenciál** | ★★★★☆ |
| **Piaci méret** | $5-10 Mrd |
| **Megtakarítás** | 90-98% |
| **Siker** | Magas |

## Piaci Megoldások

| Szoftver | Típus | Ár (hó) |
|----------|-------|---------|
| SurveyMonkey | Survey | $25-99 |
| Typeform | Interactive | $25-83 |
| Qualtrics | Enterprise | $1500+ |
| Google Forms | Basic | Ingyenes |

## FormFiller Alkalmazási Területek

### Támogatott Funkciók

| Funkció | Támogatás | Megjegyzés |
|---------|:---------:|------------|
| Választás (single/multi) | ★★★★★ | Radio, checkbox |
| Skálák (Likert, NPS) | ★★★★★ | Rating komponens |
| Szöveges válasz | ★★★★★ | Text, textarea |
| Feltételes kérdések | ★★★★★ | visibleIf logika |
| Automatikus kiértékelés | ★★★★★ | ComputedRules |
| Anonim kitöltés | ★★★★★ | Konfigurálható |
| Többnyelvű | ★★★★★ | Natív i18n |

### Speciális Funkciók

| Funkció | Támogatás |
|---------|:---------:|
| Randomizált kérdések | 🔶 Bővítéssel |
| A/B tesztelés | 🔶 Bővítéssel |
| Beágyazás (embed) | ✅ Natív |
| Offline kitöltés | 🔶 PWA |

## Bővítési Lehetőségek

| Komponens | Survey Alkalmazás | Előny |
|-----------|------------------|-------|
| **Form** | Kérdőív megjelenítés | Komplex logika |
| **Charts** | Eredmény vizualizáció | 30+ diagram típus |
| **PivotGrid** | Kereszttáblás elemzés | Aggregált statisztika |
| **DataGrid** | Válaszok listája | Szűrés, export |
| **Gauges** | NPS, CSAT kijelzés | Gyors áttekintés |

## AI Integráció

| AI Funkció | Leírás | Előny |
|------------|--------|-------|
| **Automatikus elemzés** | Válaszok összefoglalása | Gyors insight |
| **Sentiment analysis** | Szöveges válaszok | Hangulat felismerés |
| **Téma detekció** | Gyakori témák | Kategorizálás |
| **Trend felismerés** | Időbeli változás | Proaktív cselekvés |
| **Kérdés generálás** | AI javaslat | Jobb kérdőív |
| **Anomália detekció** | Szokatlan válaszminták | Minőség ellenőrzés |

## Pro/Kontra

**Előnyök:**
- 90%+ költségmegtakarítás
- Komplex feltételes logika
- Automatikus pontozás
- Self-hosted - GDPR
- Korlátlan válasz

**Hátrányok:**
- Nincs drag-drop szerkesztő (tervezett)
- Nincs beépített analitika
- Vizuális témák limitáltak

## Mikor Válaszd?

| Szcenárió | Ajánlott? |
|-----------|:---------:|
| Elégedettségi felmérés | ✅ Igen |
| Piackutatás | ✅ Igen |
| Vizsga/teszt | ✅ Igen |
| 360° feedback | ✅ Igen |
| Gyors, egyszeri felmérés | 🔶 Google Forms jobb |

## Példa: NPS Felmérés

```json
{
  "name": "npsSurvey",
  "title": "NPS Elégedettségi Felmérés",
  "items": [
    {
      "name": "npsScore",
      "type": "rating",
      "label": "Mennyire ajánlaná szolgáltatásunkat? (0-10)",
      "min": 0,
      "max": 10,
      "validationRules": [{ "type": "required" }]
    },
    {
      "name": "npsReason",
      "type": "textarea",
      "label": "Mi az oka az értékelésének?",
      "visibleIf": "npsScore !== null",
      "rows": 3
    },
    {
      "name": "improvements",
      "type": "checkboxGroup",
      "label": "Milyen területeken javíthatnánk?",
      "visibleIf": "npsScore < 7",
      "items": [
        { "value": "speed", "label": "Gyorsaság" },
        { "value": "quality", "label": "Minőség" },
        { "value": "price", "label": "Ár" },
        { "value": "support", "label": "Ügyfélszolgálat" }
      ]
    },
    {
      "name": "contactBack",
      "type": "boolean",
      "label": "Visszahívást kérek",
      "visibleIf": "npsScore < 5"
    }
  ],
  "computedFields": [
    {
      "name": "npsCategory",
      "expression": "npsScore >= 9 ? 'Promoter' : npsScore >= 7 ? 'Passzív' : 'Kritikus'"
    }
  ]
}
```

## Bővítési Lehetőségek

| Fejlesztés | Komplexitás |
|------------|:-----------:|
| Vizuális szerkesztő | Magas |
| Analitika dashboard | Közepes |
| Kérdés randomizálás | Alacsony |
| Export (SPSS, Excel) | Alacsony |

---

[← Vissza a Funkciók Összefoglalóhoz](./index.md)
