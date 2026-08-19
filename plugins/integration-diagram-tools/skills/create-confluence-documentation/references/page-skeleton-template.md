# Skeleton-template voor een integratiedocumentatie-pagina in Confluence

Gebruik deze vaste structuur voor elke pagina die deze skill aanmaakt of bijwerkt, zodat alle integratiedocumentatie in Confluence dezelfde opbouw heeft. Dit is dezelfde structuur als `functional-description-template.md` uit de documentatie-skills, aangevuld met het diagram als laatste sectie.

```
# <Integratienaam> — <richting, bijv. "SAP to Dollevoet">

## Purpose
## Trigger
## Systems and APIs involved   (tabel)
## Process flow (step by step) (genummerde lijst)
## Alternative paths and error handling
## Repetition / batch processing
## Notes / assumptions
## Diagram                      (Mermaid-codeblok, zie confluence-embedding.md voor de juiste embed-methode)
```

Bij meerdere integraties/flows op één pagina (bijv. alle flows van één klant of systeem): herhaal deze structuur per flow onder een eigen `##`-kop met het flow-nummer, en zet vooraf een korte inleiding + inhoudsopgave met links naar elke sectie (zie het Dollevoet-voorbeeld: `https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/1288077313`).

Wijk hier niet vanaf zonder het expliciet met de gebruiker te bespreken — de hele reden voor dit sjabloon is dat elke integratiepagina in Confluence er ongeveer hetzelfde uitziet, ongeacht wie of welke skill 'm heeft aangemaakt.
