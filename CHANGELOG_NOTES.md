# Changelog Notes — Documentatie-only wijziging

## Wat is er gedaan

- Er zijn **bewust geen codewijzigingen** gedaan in app-bestanden.
- Er zijn uitsluitend architectuur- en migratiedocumenten toegevoegd.

## Waarom

- **Stabiliteit eerst**: Michel wil dat de huidige app blijft werken zonder regressierisico.
- De bestaande monolith (`app.js`) bevat verweven state/render/event/business-logica; directe refactor zonder voorbereidende documentatie verhoogt risico op subtiele bugs.
- Deze documentatie creëert een veilig uitvoeringskader voor latere, kleine en controleerbare stappen.

## Expliciet niet gewijzigd

- `index.html`
- `app.js`
- `app.css`
- `sw.js`
- `manifest.webmanifest`

## Next safe actions

1. Alleen inline codecomments toevoegen om virtuele modulegrenzen zichtbaar te maken.
2. Pure read-only tests toevoegen rond bestaande business rules (duur, totals, statusafleiding).
3. Snapshot-tests toevoegen voor kritieke renderstukken zonder markupwijziging.
4. Per toekomstige stap een git tag zetten en regressiecheck doen.
5. Pas na succesvolle testdekking: kleine interne isolatiestappen uitvoeren.

## Risicobeoordeling

- Runtime-risico van deze wijziging: **nihil** (documentatie-only).
- Operatierisico: laag; team heeft nu heldere v2-doelarchitectuur zonder productie-impact.
