# Tuinlog v2 — Voorgesteld state-schema

## Doel

Een expliciet, uitbreidbaar schema dat de huidige datastructuur behoudt in betekenis, maar duidelijk splitst in:

- `entities`
- `ui`
- `settings`
- `meta`

> Dit is een voorstel voor v2. Huidige code blijft ongewijzigd.

---

## Schema-overzicht (voorstel)

```json
{
  "meta": {
    "version": 2,
    "migratedFrom": 1,
    "lastSavedAt": 0
  },
  "settings": {
    "hourlyRate": 38,
    "vatRate": 0.21,
    "locale": "nl-BE",
    "currency": "EUR"
  },
  "entities": {
    "clients": {
      "byId": {},
      "allIds": []
    },
    "products": {
      "byId": {},
      "allIds": []
    },
    "logs": {
      "byId": {},
      "allIds": [],
      "activeLogId": null
    },
    "settlements": {
      "byId": {},
      "allIds": []
    }
  },
  "ui": {
    "tab": "work",
    "nav": {
      "stack": [{ "view": "work" }],
      "transition": null
    },
    "editing": {
      "logId": null,
      "settlementId": null,
      "segmentId": null
    },
    "filters": {
      "logbook": {
        "status": "open",
        "show": false,
        "clientId": "all",
        "period": "all",
        "groupBy": "date",
        "sortDir": "desc"
      }
    }
  }
}
```

---

## Entity-definities

## `clients`

- id
- nickname
- name
- address
- createdAt
- demo (compatibiliteit)

## `products`

- id
- name
- unit
- unitPrice
- vatRate
- defaultBucket (`invoice`/`cash`)
- demo

## `logs`

- id
- clientId
- date
- createdAt
- closedAt
- note
- segments[] (`work`/`break`, start/end)
- items[] (productId, qty, unitPrice, note)
- demo

## `settlements`

- id
- clientId
- date
- createdAt
- logIds[]
- lines[] (bucket, qty, unitPrice, vatRate, product link)
- status (`draft`/`calculated`/`paid`)
- markedCalculated
- isCalculated
- calculatedAt
- invoiceAmount
- cashAmount
- invoicePaid
- cashPaid
- demo

---

## UI-state (v2)

## Tab/navigatie
- `ui.tab`: `work | money | more`
- `ui.nav.stack`: push stack met view states
- `ui.nav.transition`: `push | pop | null`

## Editing
- `ui.editing.logId`
- `ui.editing.settlementId`
- `ui.editing.segmentId`

## Filters
- `ui.filters.logbook.status`
- `ui.filters.logbook.show`
- `ui.filters.logbook.clientId`
- `ui.filters.logbook.period`
- `ui.filters.logbook.groupBy`
- `ui.filters.logbook.sortDir`

---

## Mapping: huidige localStorage -> v2 equivalent

| Huidige key in localStorage | Huidig pad | v2 equivalent | Opmerking |
|---|---|---|---|
| `tuinlog_mvp_v1` | hele payload | `meta.version=2` + opgesplitste root (`entities/ui/settings/meta`) | Eén-op-één data, ander omhulsel |
| `tuinlog_mvp_v1` | `customers[]` | `entities.clients.byId + allIds` | hernoem customer -> client |
| `tuinlog_mvp_v1` | `products[]` | `entities.products.byId + allIds` | normalisatie |
| `tuinlog_mvp_v1` | `logs[]` | `entities.logs.byId + allIds` | normalisatie |
| `tuinlog_mvp_v1` | `settlements[]` | `entities.settlements.byId + allIds` | normalisatie |
| `tuinlog_mvp_v1` | `activeLogId` | `entities.logs.activeLogId` | semantisch gelijk |
| `tuinlog_mvp_v1` | `settings` | `settings` | vrijwel direct |
| `tuinlog_mvp_v1` | `logbook` | `ui.filters.logbook` | direct met naamconversie |
| `tuinlog_mvp_v1` | `ui.editLogId` | `ui.editing.logId` | direct |
| `tuinlog_mvp_v1` | `ui.editSettlementId` | `ui.editing.settlementId` | direct |
| `tuinlog_mvp_v1` | legacy `ui.logFilter` | `ui.filters.logbook.status` | al deels gemigreerd in v1 loader |
| `tuinlog_mvp_v1` | legacy `ui.logCustomerId` | `ui.filters.logbook.clientId` | al deels gemigreerd in v1 loader |
| `tuinlog_mvp_v1` | legacy `ui.logPeriod` | `ui.filters.logbook.period` | map: `7d->week`, `90d->month`, ... |

---

## Migratieregels (status en calculatie)

## Regel 1 — Settlement status normalisatie
- Als `status` ontbreekt -> `draft`.
- Als `isCalculated=true` of `calculatedAt` bestaat -> minimaal `calculated`.

## Regel 2 — Paid afleiding
- `status=paid` wanneer:
  - invoice-bedrag > 0 en `invoicePaid=true`, én
  - cash-bedrag > 0 en `cashPaid=true`,
  - rekening houdend met nul-bedrag buckets.

## Regel 3 — Draft/calculated/paid prioriteit
- Prioriteit: `paid` > `calculated` > `draft`.
- `draft`: nog niet berekend of expliciet teruggezet.
- `calculated`: berekend maar niet volledig betaald.

## Regel 4 — Bucket defaults
- Ontbrekend `bucket` in line -> `invoice`.
- Bij cash-line `vatRate` default 0.

## Regel 5 — Legacy UI velden
- Oude velden onder `ui.*` worden bij migratie gelezen, overgezet en daarna genegeerd.

---

## Niet-doen in deze fase

- Geen herschrijven van opslagformaat in productioncode.
- Geen refactor van runtime objecttoegang.
- Geen wijziging van business rules zonder aparte validatie.
