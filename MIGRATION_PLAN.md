# Tuinlog v2 — Veilig migratieplan in 10 kleine stappen

## Uitgangspunten

- Elke stap is **klein en optioneel**.
- Na elke stap moet de app blijven werken.
- Eerst **documenteren en structureren**, pas later intern opdelen.
- We houden runtime-gedrag maximaal identiek.

---

## Stap 1 — Baseline vastzetten

**Actie**
- Maak een baseline-tag en noteer huidige functionele scope.

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-01-baseline`

---

## Stap 2 — Virtuele modulegrenzen annoteren

**Actie**
- Voeg codecomments toe (later) die secties labelen als `core/state`, `core/storage`, `ui/screens`, `domain/*`.

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-02-virtual-modules`

---

## Stap 3 — Contracten per module uitschrijven

**Actie**
- Definieer per virtuele module input/output-contracten in docs (geen code move).

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-03-contracts`

---

## Stap 4 — Selectorlaag definiëren

**Actie**
- Beschrijf pure selectors (read-only afleidingen) als aparte conceptlaag.

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-04-selectors`

---

## Stap 5 — Event intent-namen standaardiseren

**Actie**
- Beschrijf een namingconventie voor handlers/intents (bijv. `onLogStart`, `onSettlementRecalc`).

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-05-event-intents`

---

## Stap 6 — State schema v2 voorbereiden

**Actie**
- Leg v2-schema vast met entities/ui/settings/meta + mapping van oude key(s).

**Gedrag**
- **Geen gedrag wijzigen**.

**Rollback**
- `git checkout tags/v2-step-06-state-schema`

---

## Stap 7 — Read-path adapter introduceren (future)

**Actie**
- Voorzie een adapter die v1-state kan lezen als v2-viewmodel, zonder write-path te wijzigen.

**Gedrag**
- **Gedrag wijzigt licht** (alleen intern read-pad; visueel/functional gelijk beoogd).

**Rollback**
- `git checkout tags/v2-step-07-read-adapter`

---

## Stap 8 — Scherm-voor-scherm render isoleren (future)

**Actie**
- Isoleer rendering per hoofdscherm achter duidelijke functies/boundaries.

**Gedrag**
- **Gedrag wijzigt licht** (klein risico op render-volgordeverschillen).

**Rollback**
- `git checkout tags/v2-step-08-render-isolation`

---

## Stap 9 — Domeinregels expliciet centraliseren (future)

**Actie**
- Verplaats berekeningen/regels conceptueel naar `domain/work` en `domain/money` zonder functionele aanpassing.

**Gedrag**
- **Gedrag wijzigt licht** (afronding/timing-risico’s monitoren).

**Rollback**
- `git checkout tags/v2-step-09-domain-centralization`

---

## Stap 10 — Storage versiebeheer activeren (future)

**Actie**
- Introduceer `meta.version` en gecontroleerde migratiepipeline v1 -> v2.

**Gedrag**
- **Gedrag wijzigt licht** (eerste load met migratiepad).

**Rollback**
- `git checkout tags/v2-step-10-versioned-storage`

---

## Rollback-strategie (algemeen)

- Maak na elke stap een tag:
  - `v2-step-01-baseline`
  - `v2-step-02-virtual-modules`
  - ...
  - `v2-step-10-versioned-storage`
- Bij regressie:
  1. stop verdere stappen;
  2. reset naar laatste stabiele tag;
  3. herneem in kleinere sub-stapjes;
  4. documenteer oorzaak en testscenario.

Voorbeeldflow:

```bash
git tag v2-step-04-selectors
git push --tags
# regressie ontdekt
git checkout v2-step-04-selectors
```

---

## Acceptatie per stap

- App start zonder errors.
- Navigatie Werk/Geld/Meer werkt.
- Log aanmaken en afrekening openen blijft mogelijk.
- LocalStorage blijft leesbaar.
- Geen onverwachte dataverliesgevallen.
