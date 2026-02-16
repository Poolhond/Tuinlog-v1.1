# Tuinlog UI Rules — Harde UX-spec (v2 target)

## Status

Dit document is een **bindende UX-spec** voor toekomstige implementatie.
Huidige app mag blijven werken zoals nu; regels hieronder sturen komende UI-wijzigingen.

---

## 1) Iconen en betaalstatus (hard)

## Regel 1.1 — Vóór berekenen
- **Geen cash/factuur-iconen tonen** zolang een afrekening niet berekend is.

## Regel 1.2 — Na berekenen
- Toon **alleen iconen waarvoor effectief bedrag bestaat** (invoice/cash).
- Die iconen zijn **oranje** zolang niet betaald.

## Regel 1.3 — Betaald
- Iconen voor betaalde componenten zijn **groen**.

## Regel 1.4 — Alleen symbolen
- Gebruik **enkel symbolen/iconen**, geen begeleidende tekstlabels naast de statusiconen.

---

## 2) Rij-layout werklogs (hard)

Elke log-rij gebruikt exact deze visuele structuur:

```text
[TIJD links]   [PRIJS midden]   [PRODUCTEN rechts]
```

- Tijdblok links (compact, monospaced toegestaan).
- Prijs gecentreerd.
- Productindicator rechts.
- **Geen puntjes/ellipsen/extra separators** in de rij.
- Layout blijft leesbaar op small-width iPhone viewport.

---

## 3) Navigatiepatroon (hard)

## Regel 3.1 — Root tabs
- Root-navigatie bestaat uit 3 tabs:
  - **Werk**
  - **Geld**
  - **Meer**

## Regel 3.2 — Push navigation
- Details openen als **push** bovenop actieve tab.
- Back keert terug binnen dezelfde tabcontext.

## Regel 3.3 — Swipe back
- Detailschermen ondersteunen **swipe back** als primaire iOS-interactie.

---

## 4) Deviceprioriteit (hard)

- Eerste ontwerpprioriteit: **iPhone 16 Pro Max portrait**.
- Andere schermen volgen secundair via responsieve degradatie.

---

## 5) Kleursemantiek (hard)

- Oranje = berekend/openstaand.
- Groen = betaald.
- Geen ambigue statuskleuren voor betaaliconen.

---

## 6) Toetsingschecklist (implementatie-ready)

Gebruik onderstaande checks bij elke UI-PR:

- [ ] Onberekend: geen cash/factuur-iconen zichtbaar.
- [ ] Berekend: alleen iconen met bedrag zichtbaar.
- [ ] Openstaand: iconen oranje.
- [ ] Betaald: iconen groen.
- [ ] Geen tekstlabels naast iconen.
- [ ] Log-rijlayout exact: tijd links, prijs midden, producten rechts.
- [ ] Geen puntjes/separators toegevoegd.
- [ ] Push navigation werkt.
- [ ] Swipe back werkt op detail.
- [ ] Gevalideerd op iPhone 16 Pro Max portrait.
