---
name: integration-sequence-diagram-top-down
description: Genereer een Niveau 3 Integratieproces sequence diagram (Mermaid) en bijbehorende functionele beschrijving vanuit een bestaande Mulesoft- of Frends-integratie. Gebruik deze skill altijd wanneer de gebruiker vraagt om een sequence diagram, integratiediagram, functionele beschrijving van een integratie, of documentatie voor een Mulesoft-flow of Frends-proces te genereren — ook als ze alleen "diagram voor deze integratie" of "documenteer deze flow" zeggen zonder het woord "Mermaid" of "Niveau 3" te noemen. Trigger ook wanneer de gebruiker Mulesoft XML-flows, Frends process-exports/JSON, of C# Code Tasks uploadt en vraagt om deze te analyseren, te visualiseren of te documenteren. Deze skill implementeert het door het Low Code Integration Team vastgestelde Niveau 3-diagramstandaard (sequence diagrams, geen flowcharts).
metadata:
  version: "2.0.0"
---

# Niveau 3: Integratieproces sequence diagram — generator

**Versie:** 2.0.0

## Taal

**Alle output van deze skill — het Mermaid-diagram (labels, notes, comments) én de functionele beschrijving — wordt altijd in het Engels geschreven**, ongeacht de taal van de aangeleverde input, de flow-XML/JSON, of van deze skill-instructies zelf (die blijven in het Nederlands, voor het team). Dit geldt ook bij het corrigeren of aanvullen van een bestaand diagram: lever het resultaat in het Engels op, ook als het origineel Nederlandstalige labels bevat.

Uitzondering: technische identifiers die letterlijk uit de code/config komen (endpoint-paths, systeemnamen, veldnamen) worden niet vertaald — alleen de beschrijvende tekst eromheen (notes, functionele beschrijving, commentaar) is Engelstalig.

## Doel

Deze skill analyseert een bestaande integratie-implementatie (Mulesoft of Frends) en genereert automatisch:

1. Een **Mermaid sequence diagram** dat voldoet aan de standards voor Niveau 3 (zie `references/standards.md`).
2. Een **functionele beschrijving** van het integratieproces (zie `references/functional-description-template.md`).

Dit is de geautomatiseerde versie van het handmatige stappenplan uit de standards. De skill vervangt de standards niet — die blijven de bron van waarheid. Deze skill past ze toe.

## Wanneer gebruiken

- De gebruiker uploadt Mulesoft flow-XML, een Frends process-export (JSON) of C# Code Tasks en wil een sequence diagram en/of functionele beschrijving.
- De gebruiker beschrijft een integratieproces in eigen woorden en wil dit gedocumenteerd zien volgens de standards.
- De gebruiker vraagt om een bestaand Niveau 3-diagram te controleren, corrigeren of aan te vullen volgens de standards.

Als er geen bestand is geüpload maar de gebruiker wel over "de integratie" praat, vraag welk platform (Mulesoft of Frends) en vraag om de flow-configuratie/code, of laat de gebruiker de stappen in de tekst beschrijven (bron-systeem, doel-systeem(en), endpoints, tussenliggende API's/services, foutafhandeling).

## Werkwijze (stappenplan)

### Stap 1 — Verzamel input

Lees alle geüploade bestanden. Herken het platform:

- **Mulesoft**: `.xml` flow-bestanden, vaak met `<flow>`, `<sub-flow>`, `<http:listener>`, `<http:request>`, `<choice>`, `<foreach>`, `<scatter-gather>`, `<try>`/`<error-handler>`. Zie `references/mulesoft-analyse.md`.
- **Frends**: JSON process-exports of C# Code Tasks, vaak met `Trigger`, `Elements`/`Tasks`, `HTTP Request`, `Loop`, `Router`/`Condition`, `Exception Handler`. Zie `references/frends-analyse.md`.

Als het platform niet duidelijk is uit de bestandsinhoud, vraag het kort na — raad het niet.

### Stap 2 — Analyseer de flow

Doorloop de volledige flow vanaf het startpunt (trigger) tot en met de uiteindelijke response naar het initiërende systeem. Identificeer expliciet:

- **Participants**: alle systemen en API-lagen die met elkaar communiceren (bron-systeem, System API `-sa`, Process API `-pa`, Experience API `-ea`, externe systemen). Gebruik korte, consistente afkortingen — zie sectie 3.4 en 7.2 van de standards.
- **Interacties**: elke call tussen participants — method + endpoint (bijv. `POST /v1/orders`), en de bijbehorende response (statuscode, bijv. `200 OK`, `400 Bad Request`).
- **Synchroon vs asynchroon**: request/response calls zijn synchroon (`->>+` / `-->>-`); fire-and-forget events (queues, topics, pub/sub) zijn asynchroon (`-)`).
- **Logische blokken**: conditionele paden (choice/router → `alt`/`else`), optionele stappen zonder alternatief (`opt`), herhalingen over een lijst (foreach/loop → `loop`), parallelle verwerking (scatter-gather/parallel → `par`).
- **Foutafhandeling**: try/catch, error-handlers, on-error scopes → modelleer als `alt succes` / `else fout` met het bijbehorende statuscode-pad.
- **Groeperingen**: elke logische processtap (bijv. "ophalen OAuth-token", "aanroepen SAP") wordt een eigen `rect`-blok met een `note over` erboven, conform sectie 4 van de standards.

Sla géén interne transformatiestappen (DataWeave transforms, variabele-assignments, mapping-only stappen) op als aparte interacties — die zijn geen communicatie tussen systemen en horen niet in een sequence diagram thuis (zie "Veelgemaakte fouten", sectie 9 van de standards).

### Stap 3 — Genereer het Mermaid-diagram

Bouw het diagram exact volgens `references/standards.md`:

1. Begin met de vaste openingsregels uit de standards (theme/themeVariables config, sectie 3.1) — kopieer deze **letterlijk**, wijzig nooit de kleuren.
2. `sequenceDiagram` + `title Main Flow – <naam>` + `autonumber`.
3. Declareer alle `participant`s expliciet, vóór de eerste interactie.
4. Bouw de flow chronologisch op, boven naar beneden, met de juiste pijlnotatie (sectie 3.5 en 6):
   - `A ->>+ B: <actie>` voor start van een synchrone call met activatie
   - `B -->>- A: <statuscode>` voor het bijbehorende antwoord
   - `A -) B: <event>` voor asynchrone fire-and-forget berichten
5. Groepeer elke processtap in `rect rgb(235, 245, 255)` (main flow) of `rect rgb(235, 255, 235)` (uitstapjes/sub-calls naar externe systemen), met een `note over` erboven die de stap in one line samenvat.
6. Gebruik `alt`/`else`, `opt`, `loop`, `par` waar van toepassing — nooit `alt` zonder echt alternatief pad (gebruik dan `opt`), nooit `loop` voor iets dat slechts eventueel gebeurt.
7. Sluit af met de response helemaal terug naar het initiërende systeem.
8. Lijn de code uit in kolommen voor leesbaarheid.
9. Controleer tegen sectie 9 ("Veelgemaakte fouten") vóórdat je het diagram oplevert: geen flowchart-concepten (nodes/shapes/classDef), geen dubbele aanhalingstekens `"` in labels, consistente naamgeving.

Render het diagram in een artifact (`.mmd` of als mermaid-codeblok) zodat de gebruiker het direct kan controleren en in VS Code kan plakken.

### Stap 4 — Genereer de functionele beschrijving

Gebruik `references/functional-description-template.md` als structuur. De beschrijving is voor developers, testers en functioneel betrokkenen — leg uit *wat* de integratie doet en *waarom*, niet alleen de technische stappen.

### Stap 5 — Lever op

Maak een output-bestand (`.md`) met:

1. De functionele beschrijving.
2. Het Mermaid-diagram (codeblok, klaar om te kopiëren naar VS Code — zie de standards, sectie 2, stap 1: geen online Mermaid editor gebruiken i.v.m. dataveiligheid).

Wijs de gebruiker erop dat ze de code lokaal in VS Code moeten bewerken/bewaren (met de Mermaid + Mermaid Preview extensies, LF line endings) en in hun solution design moeten opslaan — dit is een afspraak in de standards, geen keuze van de tool.

## Kwaliteitscontrole vóór oplevering

Loop altijd deze checklist af voordat je het resultaat presenteert:

- [ ] Openingsregels (theme-config) letterlijk overgenomen, ongewijzigd
- [ ] `autonumber` aanwezig direct na `sequenceDiagram`/`title`
- [ ] Alle participants vooraf gedeclareerd met korte, logische afkortingen
- [ ] Elke synchrone call heeft een bijbehorende response met statuscode
- [ ] Activatie (`+`/`-`) consistent gebruikt bij synchrone calls
- [ ] Elke processtap heeft een `note over` + `rect`-groepering
- [ ] `alt` alleen bij een echt alternatief pad; anders `opt`
- [ ] `loop` alleen bij echte iteratie over een lijst/collectie
- [ ] Geen interne transformatie/mapping-stappen als aparte interacties gemodelleerd
- [ ] Geen flowchart-concepten, geen `"` in labels
- [ ] Flow volledig: van trigger tot en met de finale response naar het bron-systeem
- [ ] Alle notes, labels, commentaar en de functionele beschrijving zijn in het Engels (technische identifiers zoals endpoint-paths uitgezonderd)

## Versiebeheer

Deze skill houdt zijn eigen versienummer bij in de frontmatter (`metadata.version`) en in de leesbare `**Versie:**`-regel bovenaan dit document. Dit versienummer omvat **ook** `references/standards.md`: de standards zijn hardcoded onderdeel van deze skill (zie "Architectuur" hieronder), dus een wijziging daaraan is net zo goed een wijziging aan de skill en telt mee voor de versie-ophoging.

Dit versienummer is bovendien functioneel belangrijk voor de marktplaats-distributie: gebruikers zien en krijgen een update alleen aangeboden wanneer dit nummer omhoog gaat. Vergeet je de versie op te hogen, dan denkt de marktplaats dat er niets veranderd is en blijft iedereen op de oude versie zitten.

**Bij elke aanpassing aan deze skill wordt het versienummer verplicht opgehoogd**, ook als daar niet expliciet om gevraagd wordt. Bepaal zelf, op basis van de aard van de wijziging, of het een patch, minor of major betreft (semver):

- **Patch** (bijv. 1.0.0 → 1.0.1): tekstcorrecties, verduidelijkingen, kleine bugfixes die het gedrag niet wezenlijk veranderen.
- **Minor** (bijv. 1.0.0 → 1.1.0): een nieuwe stap, sectie, referentiebestand, of uitbreiding die achterwaarts compatibel is — bestaand gebruik blijft werken.
- **Major** (bijv. 1.0.0 → 2.0.0): een wijziging die het stappenplan, de output-structuur, of de manier waarop de skill wordt aangeroepen wezenlijk verandert, waardoor eerdere aannames over de skill niet meer kloppen.

Werk bij elke wijziging beide plekken bij (frontmatter én de leesbare regel) zodat ze nooit uit sync raken.

## Architectuur: waar leven de standards, en hoe komen updates bij gebruikers?

Er is bewust **geen live koppeling meer met Confluence**. `references/standards.md` is een hardcoded, gebundeld onderdeel van deze skill — geen aparte check, geen automatische sync, geen vragen aan de gebruiker over updaten. De skill leest dit bestand gewoon zoals elk ander referentiebestand.

Twee plekken, twee duidelijk gescheiden rollen:

- **Confluence** — blijft de plek waar het team de standards inhoudelijk bijhoudt en bediscussieert. Dit is puur een werkdocument voor mensen, geen bron waar de skill live uit leest.
- **De skill zelf (`references/standards.md`)** — een bewuste, periodieke kopie. Eén keer per zoveel tijd (bijv. elk kwartaal, of wanneer er een relevante wijziging op Confluence is doorgevoerd) kopieert iemand van het team de actuele Confluence-inhoud handmatig naar dit bestand, hoogt de skill-versie op (zie "Versiebeheer"), en publiceert de nieuwe versie naar de marktplaats (zie hieronder).

### Distributie via de marktplaats

Deze skill wordt gedistribueerd via een eigen, door het team beheerde marktplaats (vergelijkbaar met hoe bijvoorbeeld de Boomi Companion dit doet). Gebruikers voegen de marktplaats **eenmalig** toe en installeren de skill daaruit; nieuwe versies (inclusief bijgewerkte standards) verschijnen vanzelf als beschikbare update die met één klik te installeren is — geen handmatig `.skill`-bestand meer downloaden en opnieuw uploaden per gebruiker.

Voor het team betekent dit een simpel, herhaalbaar releaseproces:

1. Kopieer de actuele standards vanaf Confluence naar `references/standards.md`.
2. Werk zo nodig ook `mulesoft-analyse.md`, `frends-analyse.md`, `functional-description-template.md` of `assets/example-skeleton.mmd` bij, als de wijziging daar doorwerkt.
3. Hoog het versienummer op (patch/minor/major, zie "Versiebeheer") — dit is wat de marktplaats laat zien als "update beschikbaar".
4. Publiceer de nieuwe versie naar de marktplaats-repository.

Gebruikers hoeven zelf niets te doen behalve op "update" klikken wanneer die beschikbaar is.

## Referentiebestanden

- `references/standards.md` — het volledige standards-document (bron van waarheid voor alle syntax- en stijlregels), periodiek handmatig bijgewerkt vanaf Confluence.
- `references/mulesoft-analyse.md` — hoe Mulesoft-flow-elementen mappen naar sequence diagram-concepten.
- `references/frends-analyse.md` — hoe Frends-process-elementen mappen naar sequence diagram-concepten.
- `references/functional-description-template.md` — structuur voor de functionele beschrijving.
- `references/confluence-embedding.md` — geteste kennis over hoe Mermaid-diagrammen correct in Confluence renderen, voor toekomstig gebruik als deze skill wordt uitgebreid met direct publiceren naar Confluence.
- `assets/example-skeleton.mmd` — leeg startpunt met de verplichte openingsregels, klaar om in te vullen.

Lees `standards.md` altijd volledig door vóór het genereren van een diagram — dit bestand bevat de exacte sjablonen (sectie 5) en het volledige uitgewerkte voorbeeld (sectie 8) waar je de output tegen moet spiegelen.
