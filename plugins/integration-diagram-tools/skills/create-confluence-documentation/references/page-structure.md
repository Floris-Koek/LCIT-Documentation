# Structuur van een integratiedocumentatie-pagina in Confluence

De secties van een Confluence-pagina die deze skill aanmaakt of bijwerkt, komen **direct van** `../../shared/functional-description-template.md` (secties 1 t/m 7: Purpose, Trigger, Systems and APIs involved, Process flow, Alternative paths and error handling, Repetition/batch processing, Notes/assumptions) — gebruik géén eigen, afwijkende kopjes. Dit is bewust: zo blijft de structuur van een gegenereerde `.md`-beschrijving en de bijbehorende Confluence-pagina altijd identiek, en hoeft die structuur maar op één plek onderhouden te worden.

Confluence-specifieke aanvullingen op die structuur:

1. **Paginatitel**: `<Integratienaam> — <richting, bijv. "SAP to Dollevoet">`.
2. **Sectie 8** ("Reference to the diagram") uit de shared template wordt op de Confluence-pagina vervangen door een concrete **"Diagram"**-sectie met het daadwerkelijke Mermaid-codeblok erin — zie `confluence-embedding.md` voor de exacte, geteste embed-methode (kaal `<pre><code class="language-mermaid">`-blok, geen extensiemacro, geen handmatige `<details>`-wrapper).
3. **Meerdere integraties/flows op één pagina** (bijv. alle flows van één klant of systeem): herhaal de volledige structuur (secties 1–7 + Diagram) per flow onder een eigen `##`-kop met het flow-nummer, en zet vooraf een korte inleiding + inhoudsopgave met links naar elke sectie. Zie het Dollevoet-voorbeeld: `https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/1288077313`.

Wijk hier niet vanaf zonder het expliciet met de gebruiker te bespreken — de hele reden voor deze aanpak is dat elke integratiepagina in Confluence er ongeveer hetzelfde uitziet, ongeacht wie of welke skill 'm heeft aangemaakt, en dat er maar één plek is (`functional-description-template.md`) waar die structuur zelf gedefinieerd staat.
