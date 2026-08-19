---
name: integration-sequence-diagram-bottom-up
description: Zoek eerst uit wat de écht oorspronkelijke trigger van een Mulesoft- of Frends-integratieflow is, over de grenzen van de eigen repo heen door alle zusterrepo's in het landschap, en genereer daarna pas het Niveau 3 Mermaid sequence diagram en de functionele beschrijving. Gebruik deze skill altijd wanneer de gebruiker twijfelt over, wil verifiëren, of vraagt wie/wat een endpoint of flow écht aanroept — bijv. "wat triggert dit endpoint eigenlijk", "trace de bron van deze call", "wie roept dit nou écht aan", "bottom-up sequence diagram", of wanneer vermoed wordt dat het voor de hand liggende aanroepende systeem eigenlijk maar een relay/doorgeefluik is. Trigger ook als een eerdere aanname over de trigger van een flow onjuist bleek en gecorrigeerd moet worden. Gebruik `integration-sequence-diagram-top-down` in plaats hiervan als de trigger al met zekerheid vaststaat.
metadata:
  version: "2.0.0"
---

# Niveau 3: Integratieproces sequence diagram — bottom-up trigger-analyse

**Versie:** 2.0.0

## Taal

**Alle output van deze skill — het Mermaid-diagram (labels, notes, comments) én de functionele beschrijving — wordt altijd in het Engels geschreven**, ongeacht de taal van de aangeleverde input, de flow-XML/JSON, of van deze skill-instructies zelf (die blijven in het Nederlands, voor het team). Uitzondering: technische identifiers die letterlijk uit de code/config komen (endpoint-paths, systeemnamen, veldnamen) worden niet vertaald.

## Doel

Deze skill vult de handmatige (en de geautomatiseerde `top-down`) manier van diagrammen maken aan met een stap die er te vaak wordt overgeslagen: **verifiëren wat de echte, oorspronkelijke trigger van een flow is, vóórdat je hem tekent.**

In een landschap van veel kleine adapters (Mulesoft `-ea`/`-pa`/`-sa`, of Frends Process/Subprocess) is de meest voor de hand liggende aanroeper van een endpoint vaak niet de échte initiator, maar zelf ook maar een **relay** — een laag die een bericht alleen doorgeeft omdat een systeem daarvoor nóg weer een laag verder besloot dat het verstuurd moest worden. Een diagram dat die relay als trigger aanwijst, ziet er correct uit, wijst een onderzoeker bij een incident naar het verkeerde systeem, en herhaalt zichzelf — iedereen die het diagram leest, erft dezelfde foute aanname.

Deze skill vervangt de standards niet (`references/standards.md` blijft de bron van waarheid voor syntax en stijl), en vervangt ook `integration-sequence-diagram-top-down` niet — die skill blijft de juiste keuze zodra de trigger al vaststaat. Deze skill voegt er een verplichte trace-stap vóór aan toe.

## Wanneer gebruiken

- De gebruiker vraagt expliciet wat een endpoint, flow of API-call triggert, of wil dit laten verifiëren voordat een diagram wordt gemaakt.
- De gebruiker twijfelt aan, of wil laten controleren of, een aangenomen aanroeper wel echt de initiator is (bijv. "ik dacht dat systeem X dit aanroept, maar weet je het zeker?").
- Een eerder gemaakt diagram of eerdere aanname bleek onjuist en moet gecorrigeerd worden nadat de echte trigger is gevonden.
- De gebruiker vraagt om een "bottom-up" diagram, of vraagt naar de bron/oorsprong van een flow zonder zelf al te weten welk systeem dat is.

Gebruik in plaats daarvan `integration-sequence-diagram-top-down` wanneer de gebruiker de volledige triggerketen al aanlevert of met zekerheid kent, en alleen het diagram/de beschrijving nodig heeft.

## Werkwijze (stappenplan)

### Stap 1 — Bepaal het target

Leg het exacte startpunt van de zoekactie vast: het endpoint-pad, de bijbehorende host-property (Mulesoft, bijv. `${internal.api.elho-bartrack-ea}`), of de Frends-trigger/URL. Vraag dit na bij de gebruiker als het niet eenduidig uit de vraag blijkt — gok niet.

### Stap 2 — Trace de trigger-chain bottom-up

Volg het volledige stappenplan in `references/trigger-tracing.md`: zoek de directe aanroeper in het VOLLEDIGE landschap (niet alleen de voor de hand liggende buren), en controleer vervolgens of díe aanroeper zelf ook maar een relay is door zijn eigen lagen omhoog te volgen (Data Service → Functional Service → Business Service → Input Adapter voor Mulesoft; Subprocess → Process → Trigger voor Frends). Herhaal dit tot je een genuine origin hebt: een extern systeem dat inkomt zonder bekende aanroeper binnen het landschap, een scheduler, een queue met een identificeerbare publisher, of een bevestigde handmatige actie.

Let specifiek op routeringslogica op het punt waar een bericht het landschap binnenkomt (een `choice`/router op basis van een prefix, header of statuscode) — dit is vaak waar wordt beslist "voor wie is dit bericht bedoeld", en het is precies het soort fragiele conventie die later in "Open items for stakeholders" moet worden genoemd.

Documenteer elke schakel in de keten, inclusief pure relay-lagen — die horen in het uiteindelijke diagram en de beschrijving, expliciet gelabeld als relay.

### Stap 3 — Herbevestig of presenteer de correctie

Als de gevonden trigger afwijkt van wat de gebruiker (of een eerder document) aannam, meld dat expliciet en duidelijk vóórdat je verder gaat met tekenen — dit is vaak de belangrijkste uitkomst van de hele exercitie, niet een bijzaak.

### Stap 4 — Genereer het Mermaid-diagram

Bouw het diagram per flow exact volgens `references/standards.md` (zie ook `references/mulesoft-analyse.md` of `references/frends-analyse.md` voor de vertaling van flow-elementen naar sequence-diagram-concepten):

1. Vaste openingsregels (theme-config, sectie 3.1) letterlijk overnemen.
2. `sequenceDiagram` + `title Main Flow – <naam>` + `autonumber`.
3. Alle participants expliciet gedeclareerd vóór de eerste interactie — inclusief elke relay-laag uit de trace.
4. Chronologische opbouw met de juiste pijlnotatie; `rect rgb(235, 245, 255)` voor de hoofdflow, `rect rgb(235, 255, 235)` voor uitstapjes.
5. `alt`/`else`/`opt`/`loop`/`par` waar van toepassing.
6. Afsluiten met de response helemaal terug naar het initiërende systeem — het systeem dat in Stap 2 als genuine origin is vastgesteld, niet de relay die toevallig het dichtst bij het oorspronkelijke target ligt.

**Vóór je het diagram opmaakt: controleer elk `alt`/`else`-blok waarin een participant van buiten het blok is geactiveerd op de valkuil in `references/mermaid-activation-pitfalls.md`.** Kort samengevat: `A->>+B` activeert B (de ontvanger), `B-->>-A` deactiveert B (de verzender, niet A) — en als zowel de `if`- als de `else`-tak een gedeelde activatie van vóór het blok proberen te deactiveren, faalt Mermaid met "Trying to inactivate an inactive participant". Gebruik in dat geval gewone pijlen in de branches en één losse `deactivate <naam>` na het `end`.

### Stap 5 — Genereer de functionele beschrijving

Volg `references/functional-description-template.md`. Deze variant bevat, naast de secties uit de standaard-template, verplicht:
- Een expliciete correctie-zin per flow als de trigger anders bleek dan aangenomen (zie Stap 3).
- Een sectie "Open items for stakeholders" die risico's en losse eindjes benoemt die je onderweg tegenkwam (uitgeschakelde authenticatie, ongebruikte configuratie, placeholder-velden, fragiele routeringsconventies) — geframed als bedrijfsrisico, niet als codekritiek.

### Stap 6 — Lever op als lokaal bestand

Maak één `.md`-bestand met de functionele beschrijving en alle Mermaid-diagrammen als codeblokken. Lever dit af als bestand dat de gebruiker lokaal opent (VS Code, Mermaid + Mermaid Preview extensies, LF line endings) — **niet** als gepubliceerd/gehost artifact, conform de dataveiligheidsafspraak in `standards.md` sectie 2, stap 1 ("geen online Mermaid editor"). Wijs de gebruiker erop dat ze de code in hun solution design moeten bewaren.

## Kwaliteitscontrole vóór oplevering

- [ ] De trigger die in het diagram en de beschrijving als startpunt staat, is verifieerbaar de genuine origin (extern systeem, scheduler, queue-publisher, of bevestigde handmatige actie) — niet zomaar de eerst gevonden aanroeper
- [ ] Elke relay-laag in de keten is expliciet als relay benoemd, zowel in het diagram (`note over`) als in de functionele beschrijving
- [ ] Als de gevonden trigger afweek van de aanname, is dat expliciet en vooraan gemeld
- [ ] Openingsregels (theme-config) letterlijk overgenomen, ongewijzigd
- [ ] `autonumber` aanwezig direct na `sequenceDiagram`/`title`
- [ ] Alle participants (inclusief relay-lagen) vooraf gedeclareerd met korte, logische afkortingen
- [ ] Elke synchrone call heeft een bijbehorende response met statuscode
- [ ] Activatie (`+`/`-`) consistent gebruikt — en gecontroleerd tegen `mermaid-activation-pitfalls.md` bij elk `alt`/`else`-blok met een activatie van vóór het blok
- [ ] Elke processtap heeft een `note over` + `rect`-groepering
- [ ] `alt` alleen bij een echt alternatief pad; anders `opt`; `loop` alleen bij echte iteratie
- [ ] Geen interne transformatie/mapping-stappen als aparte interacties gemodelleerd
- [ ] Fragiele routeringsconventies (magic strings/prefixes) die je tegenkwam tijdens de trace staan in "Open items for stakeholders"
- [ ] Output is een lokaal `.md`-bestand, niet gepubliceerd naar een online/gehoste omgeving
- [ ] Alle notes, labels, commentaar en de functionele beschrijving zijn in het Engels

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
2. Werk zo nodig ook `mulesoft-analyse.md`, `frends-analyse.md`, `functional-description-template.md`, `trigger-tracing.md`, `mermaid-activation-pitfalls.md` of `assets/example-skeleton.mmd` bij, als de wijziging daar doorwerkt.
3. Hoog het versienummer op (patch/minor/major, zie "Versiebeheer") — dit is wat de marktplaats laat zien als "update beschikbaar".
4. Publiceer de nieuwe versie naar de marktplaats-repository.

Gebruikers hoeven zelf niets te doen behalve op "update" klikken wanneer die beschikbaar is.

## Referentiebestanden

- `references/trigger-tracing.md` — het stappenplan om de echte trigger van een flow te achterhalen, met zoekpatronen en het criterium voor "genuine origin". Lees dit eerst, vóór je begint met zoeken.
- `references/mermaid-activation-pitfalls.md` — de `alt`/`else`-activatie-valkuil en de fix, met voorbeeldcode. Lees dit vóór je een diagram opmaakt met een gedeelde activatie over een `alt`/`else`-blok.
- `references/standards.md` — het volledige standards-document (bron van waarheid voor alle syntax- en stijlregels, inclusief de exacte sjablonen in sectie 5 en het uitgewerkte voorbeeld in sectie 8), periodiek handmatig bijgewerkt vanaf Confluence.
- `references/mulesoft-analyse.md` — hoe Mulesoft-flow-elementen mappen naar sequence diagram-concepten.
- `references/frends-analyse.md` — hoe Frends-process-elementen mappen naar sequence diagram-concepten.
- `references/functional-description-template.md` — structuur voor de functionele beschrijving, inclusief de bottom-up-specifieke secties (trigger-correctie, open items).
- `references/confluence-embedding.md` — geteste kennis over hoe Mermaid-diagrammen correct in Confluence renderen, voor toekomstig gebruik als deze skill wordt uitgebreid met direct publiceren naar Confluence.
- `assets/example-skeleton.mmd` — leeg startpunt met de verplichte openingsregels, klaar om in te vullen.
