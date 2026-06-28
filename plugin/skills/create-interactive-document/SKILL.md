---
name: create-interactive-document
description: >
  Trigger this skill whenever the user's prompt matches any of the following patterns:

  CREATION triggers — any phrasing that expresses intent to make a new document:
  - "create a document", "create document", "I want to create a document", "I need a document"
  - "create a BRD", "write a BRD", "make a BRD", "new BRD"
  - "create a brief", "write a brief", "draft a brief"
  - "create a report", "write a report", "make a report"
  - "create a spec", "write a spec", "create a guide", "write a guide"
  - "draft a [anything]", "write a [anything]", "build a [document type]"
  - "I need a [document type] for [project/feature/initiative]"
  - Any prompt where the user describes a project or feature in enough detail that a
    formal document is the natural output

  MODIFICATION triggers — any phrasing that refers to editing an existing file:
  - "modify [filename]", "update [filename]", "edit [filename]", "change [filename]"
  - "open [filename]", "load [filename]", "make changes to [filename]"
  - "add a section to [filename]", "update the [section] in [filename]"
  - Any prompt referencing a specific filename that exists in the Tymraft repo docs/ folder
  - "I want to update [document name]", "can you edit [document name]"

  When in doubt, trigger. It is better to load this skill unnecessarily than to miss it.
---

# Create Interactive Document

This skill creates and iteratively edits Tymraft interactive HTML documents. Every document
follows a strict template from `templates/` in the repo. The document is previewed inline
after every change using the visualize tool.

## Step 1 — Determine mode: create or modify

**If the prompt references an existing filename or document name (modify mode):**
- Fetch the file from `docs/` in the `Tymraft/tymraft.github.io` repo
- Load it as the working document state
- Skip to Step 5 (iterative editing) and apply the user's requested changes immediately

**If the prompt is creating something new (create mode):**
- If the user has explicitly named a document type ("BRD", "brief", "guide", etc.), skip to Step 2
- If no type was given, show the type-picker widget using the visualize tool. Load the HTML from
  `references/type-picker.html` in this skill directory and render it with show_widget. Do not
  ask the user in text — let the widget do the asking. Wait for their response before proceeding.

## Step 2 — Load the template

Fetch the correct template from the repo:
- BRD → `https://raw.githubusercontent.com/Tymraft/tymraft.github.io/main/templates/brd.html`
- For types without a dedicated template yet, tell the user and offer to use BRD or create a
  simple structure. Do not invent a template from scratch.

Store the full template HTML as the working document state for this session.

## Step 3 — Gather missing information

Scan the template for unfilled `{{PLACEHOLDER}}` values. Ask the user for anything critical
that cannot be reasonably inferred (title, author, document ID). Make sensible defaults for
optional fields (date = today, version = 0.1, status = Draft).

Ask at most one question at a time. If the user provides enough context in their initial prompt
to fill most fields, proceed and fill what you can — only ask for what's genuinely missing.

## Step 4 — Build and preview

Fill all placeholders with real content. Replace every `{{PLACEHOLDER}}` — no placeholders
should remain in the final output. Then render the completed document inline using the
visualize tool with show_widget.

After rendering, briefly summarise what was filled in and ask if the user wants to:
- Continue editing a specific section
- Save the document to the repo
- Change the document type

## Step 5 — Iterative editing

On every subsequent user message, apply their requested changes to the working document,
then re-render the full document inline using show_widget. Always render the complete
document — never partial sections.

Template rules (enforce strictly):
- Never remove structural elements from the template (sections, cover block, ToC, top bar)
- Never change CSS variables, fonts, or colour tokens
- Never deviate from the template layout unless the user explicitly asks
- Section numbers must stay sequential
- Priority badges must use the correct classes: `priority-high`, `priority-medium`, `priority-low`
- Status badges: `badge-draft`, `badge-review`, `badge-approved`
- Timeline dots: `done`, `active`, or plain (pending)

## Step 6 — Saving to repo

When the user asks to save, commit the document to `docs/` in the Tymraft GitHub repo using
the github tool. The filename should be kebab-case derived from the document title.
Also add front matter metadata at the very top of the file:

```html
---
title: [Document Title]
type: [Document Type]
status: [Draft|In Review|Approved]
author: [Author]
date: [YYYY-MM-DD]
---
```

After saving, confirm the URL: `https://tymraft.github.io/docs/[filename]`

## Inline preview rules

- Use the visualize `show_widget` tool for every preview — never paste raw HTML into chat
- Always render the full document, not just changed sections
- Loading messages should be short and calm: "Updating the document...", "Rendering preview..."
- The widget renders at ~680px wide so scale the document preview accordingly — use the
  scaled-down CSS approach from the template preview (same structure, smaller font-sizes)
  so the full document is visible without scrolling too far

## What to read next

- `references/type-picker.html` — the inline widget shown when no document type is specified
- `references/brd-template-guide.md` — section-by-section guide to filling out a BRD
