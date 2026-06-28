---
name: create-interactive-document
description: >
  Use this skill whenever the user wants to create, write, draft, build, or update any kind of
  document — BRD, brief, guide, meeting notes, report, spec, or similar. Trigger on phrases like
  "create a document", "write a BRD", "draft a brief", "make a report", "update this doc",
  "add a section", "I need a document for", or any prompt where producing a structured document
  is the clear intent. Also trigger proactively when the user describes a project, feature, or
  initiative in enough detail that a formal document would be the natural output.
---

# Create Interactive Document

This skill creates and iteratively edits Tymraft interactive HTML documents. Every document
follows a strict template from `templates/` in the repo. The document is previewed inline
after every change using the visualize tool.

## Step 1 — Identify document type

If the user has explicitly named a document type ("BRD", "brief", "guide", etc.), skip to Step 2.

If no type was given, show the type-picker widget using the visualize tool. Load the HTML from
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
