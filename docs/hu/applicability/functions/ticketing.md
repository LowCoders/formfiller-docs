[← Vissza a Funkciók oldalra](index.md)

# Helpdesk / Ticketing

A helpdesk és ticketing funkcionalitás területén a FormFiller kiváló alapot biztosít hibabejelentés, jegykezelés és SLA követés terén.

## Értékelés

| Metrika | Érték |
|---------|-------|
| **Jelenlegi megfelelőség** | ★★★★☆ |
| **Fejlesztési potenciál** | ★★★★★ |
| **Piaci méret** | $15-25 Mrd |
| **Megtakarítás** | 60-80% |
| **Siker** | Magas |

## Piaci Megoldások

| Szoftver | Típus | Ár (ügynök/hó) |
|----------|-------|----------------|
| Zendesk | Enterprise | $19-199 |
| Freshdesk | SMB/Enterprise | $0-79 |
| Jira Service | IT Service | $20-45 |
| ServiceNow | Enterprise ITSM | $100+ |

## FormFiller Alkalmazási Területek

### Támogatott Funkciók

| Funkció | Támogatás | Megjegyzés |
|---------|:---------:|------------|
| Ticket beküldés | ★★★★★ | Komplex űrlap |
| Kategorizálás | ★★★★★ | Dinamikus mezők |
| Prioritás beállítás | ★★★★★ | Automatikus |
| Routing workflow | ★★★★★ | Beépített |
| SLA határidők | ★★★★☆ | Workflow + dátum |
| Belső megjegyzések | ★★★☆☆ | Bővítéssel |

### Nem Támogatott

- Omnichannel (email, chat, social)
- Automatikus válaszok
- Tudásbázis integráció
- CSAT/NPS automatikus
- Canned responses

## Bővítési Lehetőségek

| Komponens | Ticketing Alkalmazás | Előny |
|-----------|---------------------|-------|
| **DataGrid** | Ticket lista, keresés | Fejlett szűrés, export |
| **Diagram** | Workflow vizualizáció | Folyamat átláthatóság |
| **Charts** | SLA dashboard, KPI-k | Teljesítmény monitoring |
| **Gantt** | Ticket ütemezés | Határidő kezelés |
| **Form** | Ticket űrlap | Komplex mezők |

## AI Integráció

| AI Funkció | Leírás | Előny |
|------------|--------|-------|
| **Automatikus kategorizálás** | Ticket tartalom alapján | 90% gyorsabb routing |
| **Prioritás becslés** | Sürgősség meghatározás | Hatékonyabb kezelés |
| **Válasz javaslat** | Korábbi megoldások | Gyorsabb válaszidő |
| **Sentiment analysis** | Ügyfél hangulat | Prioritás módosítás |
| **Duplikáció detekció** | Hasonló ticketek | Összevonás |
| **NL keresés** | "Mutasd a nyitott kritikus ticketeket" | Gyors áttekintés |

## Pro/Kontra

**Előnyök:**
- Költséghatékony (70% megtakarítás)
- Testreszabható űrlapok és workflow
- Komplex routing logika
- Self-hosted - adatvédelem

**Hátrányok:**
- Nincs email ticketing
- Nincs chat integráció
- Nincs beépített riporting
- Nem multi-channel

## Mikor Válaszd?

| Szcenárió | Ajánlott? |
|-----------|:---------:|
| Belső IT helpdesk | ✅ Igen |
| Ügyfélszolgálat (egyszerű) | ✅ Igen |
| Multi-channel support | ❌ Nem |
| Enterprise ITSM | 🔶 Részben |

## Példa: Ticket Űrlap

```json
{
  "name": "supportTicket",
  "title": "Hibabejelentés",
  "items": [
    {
      "name": "category",
      "type": "select",
      "label": "Kategória",
      "items": [
        { "value": "technical", "label": "Technikai probléma" },
        { "value": "billing", "label": "Számlázási kérdés" },
        { "value": "feature", "label": "Fejlesztési igény" }
      ]
    },
    {
      "name": "priority",
      "type": "select",
      "label": "Prioritás",
      "items": [
        { "value": "critical", "label": "Kritikus" },
        { "value": "high", "label": "Magas" },
        { "value": "normal", "label": "Normál" },
        { "value": "low", "label": "Alacsony" }
      ]
    },
    {
      "name": "subject",
      "type": "text",
      "label": "Tárgy",
      "validationRules": [{ "type": "required" }]
    },
    {
      "name": "description",
      "type": "textarea",
      "label": "Leírás",
      "rows": 5,
      "validationRules": [{ "type": "required" }]
    },
    {
      "name": "attachment",
      "type": "file",
      "label": "Csatolmány",
      "accept": "image/*,.pdf,.log"
    }
  ]
}
```

## Bővítési Lehetőségek

| Fejlesztés | Komplexitás |
|------------|:-----------:|
| Email-to-ticket | Közepes |
| SLA dashboard | Alacsony |
| Canned responses | Alacsony |
| Tudásbázis link | Alacsony |

---

[← Vissza a Funkciók Összefoglalóhoz](./index.md)
