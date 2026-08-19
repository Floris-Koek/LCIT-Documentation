# Template: Functional description of an integration process

Use this structure when generating the functional description that accompanies the Niveau 3 (Level 3) sequence diagram. Audience: developers, testers, and functional stakeholders who want to understand how the integration works, both functionally and technically.

**Write the entire output in English**, even if the source flow, comments, or the person's request are in Dutch. Only literal technical identifiers (endpoint paths, system names, field names) are kept as-is; everything else — headings, explanations, notes — is in English.

Keep every section concise — this is documentation, not an essay. Use the actual system and participant names from the diagram, spelled consistently.

---

## 1. Purpose of the integration

1-3 sentences: what does this integration achieve functionally, and why does it exist? (e.g. "This integration ensures that new customer accounts created in SugarCRM are automatically registered as customers in SAP, so invoicing can happen without manual re-entry.")

## 2. Trigger

- **Type**: (HTTP event / scheduler / queue / file)
- **Source system**: which system starts the flow
- **Frequency/condition**: when or how often the flow is invoked

## 3. Systems and APIs involved

Short table or list of all participants from the diagram, with their role:

| System/API | Role in this integration |
| --- | --- |
| \<name\> | \<role, e.g. "System API that talks directly to SAP"\> |

## 4. Process flow (step by step)

Follow the same order and the same step names as the `note over` blocks in the Mermaid diagram, so the diagram and the text map 1-to-1. Per step:

- What happens functionally (no code-level detail, just the "what" and "why")
- Which endpoint(s) are used
- What the expected outcome is (happy path)

## 5. Alternative paths and error handling

Describe each `alt`/`opt` block from the diagram in plain language: under what condition does the alternative path occur, and what is the functional consequence (e.g. "If SAP returns a 400 because the VAT number is missing, the customer is not created and SugarCRM receives an error that becomes visible in the customer's CRM record.").

## 6. Repetition / batch processing (if applicable)

Describe each `loop` block: what is being iterated over, and what happens per item.

## 7. Notes / assumptions

Explicitly call out any place where an assumption was made during analysis (e.g. an ambiguous dynamic endpoint, missing error handling in the source), so a reviewer can validate it.

## 8. Reference to the diagram

Refer to the accompanying Mermaid diagram (same document or an attachment), and note that the step numbering (`autonumber`) in the diagram matches the "Process flow" section above.
