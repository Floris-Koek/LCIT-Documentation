# Bottom-up trigger tracing

Dit document beschrijft hoe je de **werkelijke** oorspronkelijke trigger van een integratieflow achterhaalt, vóórdat je het Niveau 3-diagram tekent. Dit is het kernverschil met de `integration-sequence-diagram-top-down`-skill: die skill neemt de trigger als gegeven aan (de gebruiker weet al welk systeem de flow start). Deze skill gaat ervan uit dat die aanname zelf ter discussie staat, en verifieert hem eerst.

## Waarom dit nodig is

In een landschap van veel kleine adapters/APIs (Mulesoft `-ea`/`-pa`/`-sa` of Frends Process/Subprocess) is het verleidelijk om te denken: "endpoint X wordt aangeroepen door systeem Y, want Y is de buur die er het meest voor de hand liggend uitziet." Die aanname is vaak fout, omdat Y zelf ook maar een **relay** kan zijn — een tussenlaag die het bericht alleen doorgeeft zonder zelf te beslissen dát het verstuurd moet worden.

Concreet voorbeeld uit de praktijk: bij een "processed order"-callback naar een Bartrack-adapter leek `order-pa` de initiator (het lag voor de hand: order-pa verwerkt orders, dus order-pa zal toch wel het statusbericht versturen als het klaar is?). Bottom-up tracing liet zien dat `order-pa` alleen een doorgeefluik was — de échte trigger was een **inkomende webhook van SAP** naar een heel andere adapter (`sap-sa`), die op basis van een ticketnummer-prefix besliste of het bericht naar Bartrack moest. Zonder die trace had het diagram en de functionele beschrijving een verkeerde eigenaar voor de flow aangewezen — met gevolgen voor waar je incidenten en wijzigingen aan toeschrijft.

## Stappenplan

### 1. Bepaal het target

Leg vast wat je precies aan het traceren bent: het exacte endpoint-pad, de bijbehorende host-property (bijv. `${internal.api.elho-bartrack-ea}`), of de Frends-trigger/URL. Zonder een precies target zoek je op los zand.

### 2. Zoek de directe aanroeper — in het VOLLEDIGE landschap, niet alleen de buren

Doorzoek niet alleen de repo die er "logisch" bij hoort, maar alle repos in het landschap:

- **Mulesoft**: grep naar de property-key uit de `http-request-configuration` (bijv. `internal.api.<naam>`) of het letterlijke endpoint-pad, over `*/src/main/mule/**/*.xml` én `*/src/main/resources/properties/*.yaml` van ALLE repo's.
- **Frends**: grep naar de endpoint-URL/het pad in HTTP Request Task-configuraties binnen alle process-exports (JSON).

Een match in een Data Service-laag (Mule, `4-data-services-*.xml`) of een HTTP Task-config (Frends) wijst je naar de aanroepende flow.

### 3. Vertrouw die aanroeper niet blindelings — check of HIJ zelf ook maar een relay is

Dit is de stap die het makkelijkst wordt overgeslagen, en precies de stap die de fout in het voorbeeld hierboven had voorkomen. Loop voor de gevonden aanroeper omhoog door zijn eigen lagen:

- **Mulesoft**: Data Service → Functional Service → Business Service → Input Adapter.
- **Frends**: Subprocess → Process → Trigger.

Kom je bij de Input Adapter/Trigger van die flow, stel dan vast wat het ECHTE triggertype is:

| Triggertype | Is dit een genuine origin, of moet je verder omhoog? |
| --- | --- |
| Inkomend HTTP-verzoek vanaf een ander systeem (webhook/API-call) | Nog niet klaar — zoek uit wie dat systeem is, en of dat systeem op zijn beurt ook maar reageert op iets anders |
| `<scheduler>` (Mulesoft) / Schedule Trigger (Frends) | Genuine origin — stop hier |
| Queue/topic-listener (VM, JMS, AMQP, of Frends Queue Trigger) | Nog niet klaar — zoek de publisher van die queue/dat topic elders in het landschap; die publisher is de volgende schakel |
| Bevestigde handmatige actie / UI-trigger | Genuine origin — stop hier |

### 4. Herhaal stap 3 tot je een genuine origin hebt

Blijf omhoog traceren tot je bij één van de volgende zit:
- Een extern systeem dat het landschap binnenkomt zonder dat er binnen de beschikbare repo's nog een aanroeper van dát systeem te vinden is (bijv. SAP dat zelf een callback stuurt).
- Een scheduler/cron-definitie.
- Een queue/topic met een identificeerbare publisher die zelf aan een van deze criteria voldoet.
- Een door de gebruiker bevestigde handmatige/UI-actie.

Presenteer nooit een tussenliggende relay als "de trigger". Als je de daadwerkelijke oorsprong niet met zekerheid kunt vaststellen binnen de beschikbare repo's, zeg dat expliciet in de output als aanname die de gebruiker moet bevestigen — gok niet.

### 5. Let specifiek op routeringslogica bij het punt waar een bericht het landschap "binnenkomt"

Vaak zit precies op het punt waar een extern signaal het landschap binnenkomt een `choice`/router die beslist "voor wie is dit bericht bedoeld" — bijvoorbeeld op basis van een prefix, een header, of een statuscode. Dit soort beslissingen zijn functioneel belangrijk én vaak fragiel (een magic string die, als de conventie ooit breekt, een bericht stilletjes naar de verkeerde plek stuurt). Noteer dit expliciet:
- In het diagram: als `alt`/`else` met de conditie in woorden (niet de ruwe code-conditie).
- In de functionele beschrijving: in de sectie "Open items for stakeholders" (zie `functional-description-template.md`), geframed als bedrijfsrisico, niet als codekritiek.

### 6. Documenteer élke schakel, ook de pure relays

Elke tussenlaag die je onderweg vindt — inclusief lagen die alleen doorgeven zonder zelf te beslissen — hoort in het diagram en de beschrijving, met een expliciete `note over` of zin die zegt "dit systeem is hier alleen een relay, niet de initiator." Dat ene zinnetje is vaak het nuttigste in het hele document voor iemand die dezelfde (foute) aanname had als jij aan het begin.

## Zoekpatronen — concrete startpunten

- **Property-referenties (Mulesoft)**: zoek de property-key van de `http-request-configuration` van je target (bijv. `internal.api.<naam>`) in alle `*/src/main/resources/properties/*.yaml`-bestanden én in alle `*/src/main/mule/**/*.xml`-bestanden van het volledige landschap.
- **Letterlijke pad-fragmenten**: grep naar het endpoint-pad zelf (bijv. `/internal/v1/events/processed-order`) over alle sibling-repo's — een match in een `4-data-services-*.xml` of een Frends HTTP Task-config is je aanroeper.
- **Niet vinden bij de "logische" buren?** Verbreed de zoekopdracht naar de hele repo-root, niet alleen repos die thematisch voor de hand liggen — de echte aanroeper zit soms in een repo waarvan de naam geen enkele hint geeft (zoals SAP's ticket-callback die in `sap-sa` landt, niet in de Bartrack- of order-adapter waar je het zou verwachten).

## Wanneer je klaar bent met traceren

Pas als de volledige keten van genuine origin tot en met het uiteindelijke target vaststaat, ga je verder met het tekenen van het diagram volgens `standards.md` (en `mulesoft-analyse.md`/`frends-analyse.md` voor de element-vertaling) en het invullen van `functional-description-template.md`.
