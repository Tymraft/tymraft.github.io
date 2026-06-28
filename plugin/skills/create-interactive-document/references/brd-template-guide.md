# BRD Template Guide

Section-by-section reference for filling out `templates/brd.html`.

## Top bar

| Field | How to fill |
|-------|-------------|
| `{{FILE_PATH}}` | The repo path where the file will be saved, e.g. `docs/my-brd.html` |
| Badge class | `badge-draft` (default), `badge-review`, or `badge-approved` |
| Version | Start at `v0.1`. Increment minor for edits, major for approved versions |

## Cover block

| Placeholder | Guidance |
|-------------|----------|
| `{{DOCUMENT_TITLE}}` | Short, specific. E.g. "Mobile App Redesign" not "Project" |
| `{{DOCUMENT_SUBTITLE}}` | One sentence describing what this BRD covers and why |
| `{{DOCUMENT_ID}}` | Format: `BRD-YYYY-NNN` e.g. `BRD-2026-002` |
| `{{AUTHOR}}` | Full name of the document owner |
| `{{DATE}}` | ISO format: `YYYY-MM-DD` or friendly: `28 Jun 2026` |

## Section 1 — Overview

Two paragraphs:
1. The problem or opportunity — what is happening now that requires this document?
2. The proposed solution — what will be built or changed, and where?

The callout (`{{SCOPE_NOTE}}`) is optional — use it for important caveats or to point to
related documents. Remove the callout block if not needed.

## Section 2 — Objectives

Bullet list of 3–6 concrete, measurable goals. Start each with a verb. Avoid vague language
like "improve" without a qualifier.

## Section 3 — Scope

**In scope**: What this project will deliver. Be specific.
**Out of scope**: Explicitly name things that might be assumed but won't be covered. This
prevents scope creep and sets expectations.

## Section 4 — Requirements

### Functional requirements table

- IDs: `FR-01`, `FR-02`, ... (sequential, zero-padded)
- Requirement text: Start with "The system/user/product shall..."
- Priority: Use `priority-high` (must-have), `priority-medium` (should-have), `priority-low` (nice-to-have)

### Non-functional requirements

Cover at minimum: Performance, Compatibility, Maintainability, Accessibility.
Add Security, Scalability if relevant.

## Section 5 — Stakeholders

List every person or team who has a stake in the outcome. Include:
- Project Owner / Sponsor
- End Users
- Technical leads
- Any external parties

## Section 6 — Timeline

Timeline dot states:
- `done` (green tick) — completed milestones
- `active` (blue dot) — current milestone
- plain (grey dot) — future milestones

Use specific dates where known. Use "TBD" or "Q3 2026" for uncertain future dates.

## Section 7 — Risks

The warning callout is for the single most critical risk. The table covers all risks.

Likelihood and Impact columns use the priority badge classes:
- `priority-low` = Low
- `priority-medium` = Medium  
- `priority-high` = High

## Section 8 — Approvals

List all required approvers. Status starts as `badge-review` (Pending).
Update to `badge-approved` with the approval date when signed off.
