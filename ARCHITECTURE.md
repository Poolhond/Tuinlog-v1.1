# Tuinlog v2 — Architectuurplan (zonder refactor)

## Doel en randvoorwaarden

- **Doel**: de huidige app stabiel houden en tegelijk een duidelijk pad voorzien naar een schaalbare architectuur.
- **Harde randvoorwaarde**: **geen codeverplaatsing of gedragswijziging nu**; dit document beschrijft alleen een toekomstige modulegrens op basis van de bestaande `app.js`.
- **Bron van waarheid vandaag**: één monolith (`app.js`) met state, business rules, rendering, events en localStorage in hetzelfde bestand.

---

## Hoe de huidige monolith werkt (as-is)

## 1) Data en state

- De volledige applicatiestatus wordt in één state-object opgebouwd via `defaultState()`.
- Kerncollecties:
  - `customers`
  - `products`
  - `logs`
  - `settlements`
- Extra secties:
  - `settings` (uurtarief, btw)
  - `activeLogId`
  - `ui` (edit-modus flags)
  - `logbook` (filters/sortering/groepering)
- Een tweede runtime-ui-state leeft buiten persisted state in `const ui` (o.a. `navStack`, `transition`).

## 2) Storage en migraties

- Persistency gebeurt via één localStorage-key: `tuinlog_mvp_v1`.
- `loadState()` laadt JSON, vult defaults aan, voert kleine migraties/normalisaties uit en borgt compatibiliteit met oudere UI-velden.
- `saveState()` schrijft de volledige state terug als JSON.

## 3) Rendering

- Centrale entrypoint: `render()`.
- Root-tab rendering splitst impliciet in:
  - `renderLogs()`
  - `renderSettlements()`
  - `renderMeer()`
- Detail/sheet rendering gebeurt via `renderSheet()` + specifieke detailrenderers.
- Topbar en navigatievisuals worden apart aangestuurd via `renderTopbar()` en `updateTabs()`.

## 4) Events en interactie

- Veel event handlers worden direct op DOM-elementen gekoppeld in dezelfde file.
- Acties volgen typisch dit patroon:

```text
[User event]
   -> [Handler in app.js]
      -> [State mutatie]
         -> [saveState()]
            -> [render() of renderSheet()]
```

- Navigatie gebruikt stack-gebaseerde push/pop via `ui.navStack`:

```text
setTab(root)
  -> navStack = [root]
pushView(detail)
  -> navStack.push(detail)
popView()
  -> navStack.pop()
```

---

## Virtual modules voor v2 (mapping, niet implementeren)

> Belangrijk: dit zijn **conceptuele modules**. De huidige functies blijven voorlopig waar ze zijn.

## `core/state`

**Verantwoordelijkheid**
- Definitie en lifecycle van app-state.
- Normalisatie en validatie van state na load/mutaties.

**Input**
- Raw state uit storage.
- Mutatie-intenties vanuit events/domain.

**Output**
- Geldige in-memory state voor render/domain.

**Functies die nu vermoedelijk hierbij horen**
- `defaultState()`
- `ensureUIPreferences()`
- `ensureCoreProducts()`
- `ensureStateSafetyAfterMutations()`
- Helpers rond `state.ui` edit-flags.

---

## `core/storage`

**Verantwoordelijkheid**
- Opslaan/laden van state + versiebeheer/migraties.
- Import/export/clear flows.

**Input**
- Volledige state + versiecontext.

**Output**
- Persisted JSON in localStorage.
- Gemigreerde state bij load.

**Functies die nu vermoedelijk hierbij horen**
- `STORAGE_KEY`
- `loadState()`
- `saveState()`
- LocalStorage import/export/reset routines.

---

## `ui/screens`

**Verantwoordelijkheid**
- Alle presentatie en schermspecifieke rendering.
- Topbar/tabbar/detailpagina, schermtransities, iconografie.

**Input**
- Current view (`ui.navStack`), domain-output, afgeleide viewmodels.

**Output**
- DOM updates + klikpunten voor events.

**Functies die nu vermoedelijk hierbij horen**
- `render()`
- `renderTopbar()`, `updateTabs()`, `viewTitle()`
- `renderLogs()`, `renderSettlements()`, `renderMeer()`
- `renderSheet()` + detailrenderers (`renderLogSheet`, `renderSettlementSheet`, ...)
- `renderSettlementStatusIcons()`

---

## `domain/work`

**Verantwoordelijkheid**
- Werklog lifecycle: starten, pauzeren, stoppen, segmenten, duur.
- Productregels binnen werklogs.

**Input**
- Log events (start/pause/stop/edit).
- Productkeuze en quantities.

**Output**
- Bijgewerkte `logs` + afgeleide duur/summary.

**Functies die nu vermoedelijk hierbij horen**
- `startWorkLog()`
- `openSegment()`, `closeOpenSegment()`, `currentOpenSegment()`
- `sumWorkMs()`, `sumBreakMs()`, duurformatters
- `addProductToLog()`
- Logstatus-afleiding helpers.

---

## `domain/money`

**Verantwoordelijkheid**
- Afrekening-opbouw uit logs.
- Totalen, btw, cash/factuur buckets, betaaldstatus.

**Input**
- Gekoppelde logs, lines, settings/btw-tarieven.

**Output**
- Settlement lines/totals/status.

**Functies die nu vermoedelijk hierbij horen**
- `computeSettlementFromLogsInState()` / `computeSettlementFromLogs()`
- `settlementTotals()` / `getSettlementTotals()`
- `syncSettlementAmounts()`
- `isSettlementPaid()`
- `ensureDefaultSettlementLines()`
- toggles rond calculated/invoicePaid/cashPaid.

---

## `domain/clients`

**Verantwoordelijkheid**
- Klantgegevens, klantdetail en referentiële koppeling met logs/settlements.

**Input**
- CRUD-events op klanten.

**Output**
- Gevalideerde klantenlijst en klant-specifieke overzichten.

**Functies die nu vermoedelijk hierbij horen**
- `getCustomer()`, `cname()`
- `renderCustomersSheet()`
- `renderCustomerSheet(id)`
- klantselectie in log/settlement flows.

---

## `domain/products`

**Verantwoordelijkheid**
- Productcatalogus, defaults en prijs/unit/bucket-standaarden.

**Input**
- Product CRUD + referentie vanuit logs/settlements.

**Output**
- Productenlijst + resolved producteigenschappen in business flows.

**Functies die nu vermoedelijk hierbij horen**
- `getProduct()`, `pname()`
- `preferredWorkProduct()`
- `ensureCoreProducts()`
- `renderProductsSheet()`
- `renderProductSheet(id)`

---

## iPhone-first UX target (toekomstig, nog niet doorgevoerd)

Deze principes zijn **target-state voor v2**, geen directe code-opdracht:

- Root-tab model met **Werk / Geld / Meer** als primaire entrypoints.
- **Push navigation** voor detailschermen bovenop de actieve tab.
- **Swipe back** als natuurlijke terug-actie op detailniveau.
- Eénhandig gebruik en duidelijke visuele statuscodes (kleur + iconen).
- Layoutoptimalisatie voor **iPhone 16 Pro Max in portrait** als design-basis.

---

## Dataflow (huidige en gewenste v2 denkwijze)

```text
User event
  -> Event handler
    -> Domain regel (work/money/clients/products)
      -> State update (core/state)
        -> Persist (core/storage)
          -> Render screen (ui/screens)
```

In de huidige app gebeurt dit al functioneel, maar nog binnen één bestand.
