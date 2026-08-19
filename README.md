# claude-ai-skills

A Claude plugin marketplace maintained by the Low Code Integration Team, distributing skills that analyze existing Mulesoft and Frends integrations and automatically generate standardized Level 3 sequence diagrams (Mermaid) and functional descriptions, in line with our team's documentation standards.

This repo is the source for our own marketplace (similar in spirit to, for example, the Boomi Companion marketplace): users add the marketplace once and from then on get one-click updates to these skills, instead of manually downloading and re-uploading `.skill` files.

## Contents

```
claude-ai-skills/
└── skills/
    ├── integration-sequence-diagram-top-down/
    └── integration-sequence-diagram-bottom-up/
```

Each skill tracks its own version number inside `SKILL.md` itself: in the frontmatter (`metadata.version`) and as a human-readable line right under the title (`**Version:** 2.0.0`). Every change to a skill — including a change to the bundled standards, see below — bumps this version following semver (e.g. `2.1.0` for a new step or section, `2.0.1` for a small text fix). This version number isn't just bookkeeping: it's what the marketplace uses to offer users an update. Folder and file names themselves stay unversioned.

### `integration-sequence-diagram-top-down`

The default skill for generating a Level 3 sequence diagram and functional description from an existing Mulesoft or Frends integration. Use this when the trigger of the flow is already known and only the diagram/description is still needed.

### `integration-sequence-diagram-bottom-up`

Extends the top-down skill with a mandatory tracing step beforehand: first establishes the true, original trigger of a flow (across repository boundaries, through relay layers such as Mulesoft `-ea`/`-pa`/`-sa` or Frends Subprocess/Process/Trigger), and explicitly flags a correction if the assumed trigger turns out to be wrong. Use this when there's doubt about who/what actually calls a flow, or when an earlier assumption needs to be corrected.

Both skills use the same standards and output conventions (English-language output, the same Mermaid style); they differ only in the up-front analysis step.

## Architecture: where do the standards live?

**No live connection to Confluence.** Each skill contains a hardcoded `references/standards.md` that is simply read as a bundled reference file — no check, no sync, no prompting the user about updates. This is a deliberate choice: simpler, more predictable, and no dependency on a connector at runtime.

Two places, two roles:

1. **Confluence** — remains the working document where the team maintains the standards' content (Low Code Integration → Diagram standaardisatie → Niveau 3).
2. **This repo** — where the standards are periodically, manually, copied over from Confluence into `skills/<name>/references/standards.md`, followed by a version bump and a release to the marketplace. See "Updating the standards" below.

## Installing a skill

**Via the marketplace (recommended):**

1. Add this marketplace once in Claude.
2. Install the skill(s) you want from it.
3. Future versions (including updated standards) show up automatically as an update — one click, no reinstall needed.

**Manual (alternative):** download the `.skill` file, or clone this repo and grab the folder under `skills/<name>`, and install it using the usual custom-skill workflow. Note: this route doesn't get automatic updates — which is exactly why the marketplace is the preferred path.

## Updating the standards

This is a **deliberate, manual action**, done on a fixed cadence (e.g. quarterly) or whenever a relevant change lands on Confluence — never automated:

1. Make sure the change has first been made at the source: the Confluence page "Niveau 3: Integratieproces sequence diagram".
2. Copy the updated content into `skills/<name>/references/standards.md` in both skill folders (top-down and bottom-up share the same standards).
3. Check whether the change also affects `mulesoft-analyse.md`, `frends-analyse.md`, `functional-description-template.md`, or `assets/example-skeleton.mmd`, and update those where needed.
4. Bump the version number in `SKILL.md` (both the frontmatter and the readable `**Version:**` line) — this is what the marketplace uses to offer the update.
5. Validate and package the skill again (`.skill` file).
6. Commit and push to this repo, and publish the new version to the marketplace. Briefly describe what changed in the commit message.

Users then don't need to do anything except accept the offered update.

## Adding a new skill

Missing functionality, or need a whole new type of diagram/documentation? Let Floris or Sophie know, or add a new folder under `skills/` yourself following the same structure (`SKILL.md`, `references/`, `assets/`), and add it to the marketplace.

## Roles

- **Floris & Sophie** — development and maintenance of these skills.
- **Rudy de Wit** — supervisor & assessor.
- **Jan Willem van Doornspeek** — assessor.

## Related documentation

- Confluence: [Diagram standaardisatie](https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/664436738/Diagram+standaardisatie) — all diagram levels (1 through 4).
- Confluence: [Niveau 3: Integratieproces sequence diagram](https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/908328961/Niveau+3+Integratieproces+sequence+diagram) — the working document the standards are periodically copied from.
- Confluence: [Automatisch Genereren](https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/1249738757/Automatisch+Genereren) — explainer on these skills for the wider team.
