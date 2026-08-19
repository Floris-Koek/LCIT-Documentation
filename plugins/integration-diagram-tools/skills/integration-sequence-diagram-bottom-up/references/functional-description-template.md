# Template: Functional description (bottom-up variant)

Gebruik deze structuur pas nadat de triggerketen van elke flow volledig is vastgesteld via `trigger-tracing.md`. Doelgroep: zowel niet-technische/business-lezers als technische lezers — schrijf platte taal eerst, techniek/diagram erna.

**Schrijf de volledige output in het Engels**, ook als de bron-flows, comments, of het verzoek van de gebruiker in het Nederlands zijn. Alleen letterlijke technische identifiers (endpoint-paths, systeemnamen, veldnamen) blijven ongewijzigd — headings, uitleg, notes zijn Engelstalig.

---

## 1. Purpose

1-3 sentences: what business capability does this integration provide, and for whom (which external partner, department, or system)?

## 2. Systems involved

Table: `System | Role`. Include EVERY system found while tracing the chain — including pure relays. Mark relay-only systems explicitly in their role description, e.g. *"Acts only as a relay in Flow 4, not the initiator."* This is the one place a reader scanning quickly will still notice the correction.

## 3. One section per distinct flow

For each flow, in this order:

1. **In plain terms** — a short paragraph a non-technical reader can follow: what triggers this, what happens, what comes back.
   - If the naive/assumed trigger turned out to be wrong during tracing, say so explicitly, e.g.: *"It's tempting to assume `order-pa` starts this — it processes orders, after all. Tracing the outbound call shows `order-pa` only relays the message; the real origin is an inbound webhook from SAP into `sap-sa`."* This sentence is often the single most useful line in the document, because it's exactly the assumption a future reader is most likely to repeat.
   - Call out any business rule enforced along the way (validation limits, required fields, routing conditions) in plain language.
2. **Diagram** — the Mermaid sequence diagram for this flow, built per `standards.md` (opening config, `title`, `autonumber`, explicit participants, `rect` colouring, `alt`/`opt`/`loop`/`par` where applicable) and checked against `mermaid-activation-pitfalls.md` if it contains an `alt`/`else` block with a shared activation.

## 4. Business rules worth knowing

A flat list of validation rules, thresholds, or non-obvious behaviour a business stakeholder should know without reading code (e.g. "an order may contain at most 10 remarks, each under 70 characters — violations are rejected before reaching the order system").

## 5. Open items for stakeholders

Anything noticed along the way that's a risk or loose end, framed as a **business-relevant consequence**, not a code-review nitpick:

- Disabled/commented-out authentication ("this internal endpoint currently accepts requests without a login check — a deliberate rollout decision that should be revisited").
- Unused or half-configured resources (queues, connectors) left over from earlier design decisions.
- Placeholder/not-implemented response fields the caller might rely on.
- **Fragile routing conventions** found while tracing — a magic string, header value, or prefix silently deciding which downstream system a message goes to. These are worth flagging even when they currently work correctly, because the failure mode is silent (a message goes to the wrong place, or nowhere, without an error) rather than loud.

## 6. Reference to the diagrams

Note that each flow's diagram is delivered as a Mermaid code block in the same document, ready to paste into VS Code (Mermaid + Mermaid Preview extensions, LF line endings) per the delivery rules in `standards.md` — never published to a hosted/online Mermaid editor, for the data-safety reason stated in that standard.
