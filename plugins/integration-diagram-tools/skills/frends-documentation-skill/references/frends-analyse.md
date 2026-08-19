# Frends-processen analyseren voor het Niveau 3 sequence diagram

Dit document beschrijft hoe je elementen uit een Frends-proces (JSON-export van een Process, of een C# Code Task) herkent en vertaalt naar het Niveau 3 Mermaid sequence diagram. Gebruik dit samen met `../../shared/standards.md`. Raadpleeg bij twijfel over Frends-specifieke concepten (Tasks, Agents, Environment Variables, expressies) ook de `fc-integration:frends-ipaas-developer`-skill, indien beschikbaar.

## 1. Waar te beginnen

Zoek in het process-export (JSON) naar de **Trigger**:

- `HTTP Trigger` / `HTTP Request Trigger` → dit is de start-interactie: `<bron-systeem> ->>+ <frends-proces>: <METHOD> <path>`.
- `Schedule Trigger` → self-call zoals in sjabloon 5.1: `note over <proces>: Scheduler` gevolgd door `<proces> ->>+ <proces>: Start processing` ... `<proces> ->>- <proces>: End processing`.
- `Queue Trigger` / event-gebaseerde trigger → asynchrone start: `<bron-systeem> -) <frends-proces>: <event>`.

## 2. Participants herkennen

In Frends is elk **Process** (of duidelijk afgebakende Subprocess die naar een ander extern systeem praat) een participant. Herken:

- Het aanroepende/bron-systeem (van de Trigger).
- Het Frends-proces zelf (of de relevante Subprocess als het proces uit meerdere logisch gescheiden Subprocessen bestaat — behandel elke Subprocess als eigen participant als dat de leesbaarheid verbetert, anders als interne stap binnen dezelfde participant).
- Elk extern doelsysteem waarmee een Task communiceert (SAP, een REST-API, een database, een file-share, etc.).

Gebruik dezelfde afkortingsconventie als bij Mulesoft waar van toepassing (bijv. `-api` suffix voor een API-laag), of de bestaande naamgeving uit de Frends Environment als die er is.

## 3. Frends-elementen → sequence diagram-concepten

| Frends-element | Sequence diagram-vertaling |
| --- | --- |
| **HTTP Request Task** naar extern systeem/API | Synchrone call: `A ->>+ B: <METHOD> <path>` gevolgd door `B -->>- A: <status>` |
| **Subprocess-aanroep** die zelf externe calls doet | Eigen participant-lijn, of geneste `rect`-groepering binnen de aanroepende stap (zie het geneste voorbeeld in de standards, sectie 8) |
| **Router / Condition-element** (If/Switch) | `alt <conditie 1> ... else <conditie 2> ... end` |
| **Loop-element** (ForEach over een collectie) | `loop <omschrijving iteratie> ... end` |
| **Parallel/Concurrent execution** | `par <flow 1> ... and <flow 2> ... end` |
| **Exception Handler** / try-catch rond een Task | `alt succes ... else fout (<statuscode>) ... end` |
| **Queue Task** (verzenden naar/lezen van een message queue) | Asynchrone message: `A -) B: <event>` |
| **Code Task (C#)** die alleen data transformeert/mapt (geen externe call) | **Niet modelleren** als interactie — interne verwerking. Alleen relevant als het een functionele beslissing/validatie betreft; modelleer dat dan als `opt` of `note`, niet als call naar een participant. |
| **Code Task (C#)** die wél een externe library/API aanroept (bijv. een custom NuGet Task die naar een extern systeem praat) | Wel als synchrone/asynchrone call modelleren, zoals hierboven — behandel de aanroep zoals een normale Task-call |

## 4. Endpoints en labels

Gebruik de daadwerkelijke HTTP-method + path (of het queue-/topic-naam voor async) uit de Task-configuratie:

```
frends_proces ->>+ sap_api: POST /api/v1/orders
sap_api -->>- frends_proces: 200 OK
```

## 5. Wat je NIET in het diagram zet

- Environment Variable-lookups en configuratie.
- Logging-Tasks, tenzij expliciet gevraagd.
- Pure data-mapping/transform Code Tasks zonder externe communicatie.

## 6. Onduidelijke of ontbrekende informatie

Als een Task een dynamisch URL-adres gebruikt (samengesteld uit variabelen) of de doel-API niet eenduidig uit de export blijkt, benoem dit als aanname in de functionele beschrijving en vraag de gebruiker om bevestiging in plaats van te gokken.
