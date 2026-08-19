---
name: create-confluence-documentation
description: Publiceer integratiedocumentatie (functionele beschrijving + Mermaid sequence diagram) als een pagina in Confluence, volgens het vaste teamsjabloon en de geteste Mermaid-embedding-methode. Gebruik deze skill wanneer de gebruiker vraagt om gegenereerde documentatie (van `mulesoft-documentation-skill`, `frends-documentation-skill`, of aangeleverde tekst/diagrammen) naar Confluence te zetten, te publiceren, of daar bij te werken — bijv. "zet dit op Confluence", "maak hier een Confluence-pagina van", "publiceer deze documentatie". Trigger niet automatisch na het genereren van een diagram — alleen op expliciet verzoek van de gebruiker.
metadata:
  version: "1.0.0"
---

# Documentatie publiceren naar Confluence

**Versie:** 1.0.0

## Doel

Deze skill neemt bestaande integratiedocumentatie (functionele beschrijving + Mermaid sequence diagram, meestal net gegenereerd door `mulesoft-documentation-skill` of `frends-documentation-skill`) en zet die als pagina in Confluence, volgens een vast sjabloon en met correct renderende Mermaid-diagrammen.

Deze skill is bewust losgetrokken uit de twee documentatie-skills: het wegschrijven naar Confluence was daar identiek, dus staat het nu op één plek in plaats van dubbel.

## Wanneer gebruiken

- De gebruiker vraagt expliciet om net gegenereerde documentatie naar Confluence te publiceren.
- De gebruiker vraagt om een bestaande Confluence-pagina met integratiedocumentatie bij te werken.
- De gebruiker levert zelf een functionele beschrijving + diagram aan (niet per se via de andere twee skills) en wil dat in Confluence-vorm.

**Trigger dit nooit automatisch** na het genereren van een diagram door een van de andere skills — alleen op expliciet verzoek. Documentatie genereren en documentatie publiceren zijn bewust twee losse stappen.

## Harde regels (niet onderhandelbaar)

Deze drie regels zijn eerder expliciet met het team afgesproken en gelden altijd, zonder uitzondering:

1. **Nooit zelf iets plaatsen of overschrijven zonder te vragen.** Toon altijd eerst wat er gaat gebeuren (titel, locatie, of het een nieuwe pagina of een update van een bestaande is) en wacht op expliciete bevestiging voordat je `createConfluencePage` of `updateConfluencePage` aanroept.
2. **Nooit zelf de locatie bedenken.** Je mag een plek voorstellen (bijv. "onder 'Diagram standaardisatie', net als de vorige testpagina"), maar de gebruiker bepaalt de uiteindelijke ouderpagina/space. Vraag dit na als het niet is aangegeven.
3. **Altijd het skeleton-sjabloon gebruiken** (`references/page-skeleton-template.md`) voor de opbouw van de pagina, zodat alle integratiepagina's in Confluence dezelfde structuur hebben.

## Werkwijze (stappenplan)

### Stap 1 — Verzamel de content

Gebruik de functionele beschrijving en het Mermaid-diagram die al in het gesprek staan (van `mulesoft-documentation-skill`/`frends-documentation-skill`, of rechtstreeks aangeleverd door de gebruiker). Vraag na als een van beide ontbreekt of onvolledig is — genereer zelf geen nieuwe documentatie, dat is niet de taak van deze skill.

### Stap 2 — Bepaal titel en locatie, en stel dit voor

Stel een paginatitel voor (bijv. `<Integratienaam> — Level 3 Sequence Diagram`), en stel een locatie voor (ouderpagina + space) op basis van wat bekend is over hoe het team Confluence indeelt — maar **beslis dit niet zelf**. Vraag de gebruiker om de locatie te bevestigen of te corrigeren. Vraag ook expliciet: gaat het om een **nieuwe** pagina, of het **bijwerken** van een bestaande?

### Stap 3 — Bouw de pagina op volgens het sjabloon

Volg `references/page-skeleton-template.md` exact. Zet het Mermaid-diagram neer volgens `references/confluence-embedding.md`: een kaal `<pre><code class="language-mermaid">`-blok (of ```` ```mermaid ```` -fence in markdown-content), **zonder** aparte extensiemacro en **zonder** eigen `<details>`-wrapper eromheen — dat codeblok rendert in Confluence al vanzelf als diagram mét ingebouwde toggle voor de ruwe code.

### Stap 4 — Presenteer het voorstel en vraag bevestiging

Toon de gebruiker een korte samenvatting: titel, locatie, of het nieuw is of een update, en wat er ongeveer op de pagina komt te staan. Vraag expliciete bevestiging ("Zal ik deze pagina nu aanmaken/bijwerken?") vóórdat je de Confluence-API aanroept. Ga pas door na een duidelijk "ja".

### Stap 5 — Publiceer

Roep pas nú `createConfluencePage` (nieuwe pagina) of `updateConfluencePage` (bestaande pagina) aan via de Atlassian/Confluence-connector, met de opgebouwde content uit Stap 3. Meld na afloop de link naar de pagina, en vraag de gebruiker om zelf te controleren of het diagram goed rendert (dat kan deze skill niet zelf visueel verifiëren).

## Referentiebestanden

- `references/page-skeleton-template.md` — het vaste sjabloon voor de opbouw van een integratiedocumentatie-pagina.
- `references/confluence-embedding.md` — geteste, bevestigde kennis over hoe je een Mermaid-diagram correct in Confluence laat renderen, inclusief wat je juist *niet* moet doen (geen extra extensiemacro, geen handmatige `<details>`-wrapper).
