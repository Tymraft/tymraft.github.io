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

This skill creates and iteratively edits Tymraft interactive HTML documents. Documents are
built using the **Jekyll includes system** — Claude writes a small data file (~1–2KB) and
Jekyll assembles the final page from reusable components. Claude never reads or writes the
full template HTML.

Use plain, non-technical language at all times. Avoid technical terms — use everyday words
instead. For example:
- "save" not "commit"
- "publish to the site" not "push to the repo"
- "stored on the site" not "in the repo"
- "document name" not "filename" or "kebab-case"
- "ready" or "up to date" not "synced"
- "make a change" not "apply a diff"

## Step 1 — Determine mode: create or modify

**If the prompt references an existing document name (modify mode):**
- Fetch the file from `docs/` in the `Tymraft/tymraft.github.io` repo
- Load it as the working document state
- Skip to Step 5 (iterative editing) and apply the user's requested changes immediately

**If the prompt is creating something new (create mode):**
- If the user has explicitly named a document type ("BRD", "brief", "guide", etc.), skip to Step 2
- If no type was given, show the type-picker widget using the visualize tool. Load the HTML from
  `references/type-picker.html` in this skill directory and render it with show_widget. Do not
  ask the user in text — let the widget do the asking. Wait for their response before proceeding.

## Step 2 — Identify the document type and layout

Match the requested document type to its Jekyll layout:

| Document type | layout | Available includes |
|---|---|---|
| BRD | `brd` | cover, section, callout, timeline-item (see below) |

For types without a layout yet, tell the user in plain terms and offer to use the BRD layout
or build a simple version. Do not invent a template from scratch.

**Do not fetch any template file.** The layout handles all CSS, fonts, and structure
automatically. Claude only writes the document's content.

## Step 3 — Gather missing information

Ask the user for anything critical that cannot be reasonably inferred:
- Document title
- Author
- Document ID (e.g. BRD-2026-001)
- A brief description (subtitle)

Make sensible defaults for optional fields: date = today, version = 0.1, status = draft.

Ask at most one question at a time. If the user provides enough context in their initial prompt
to fill most fields, proceed and only ask for what's genuinely missing.

Use plain language:
- "What would you like to call this document?" not "What is the document title?"
- "Who is writing this?" not "Who is the author?"
- "What is this document for?" not "Provide a subtitle or description."

## Step 4 — Build the document file

Compose the document as a small Jekyll file. This is the **only thing Claude writes** — no CSS,
no SVG, no boilerplate.

### BRD file structure

```
---
layout: brd
title: "[Document Title]"
file_path: "[filename].html"
status: draft
version: "0.1"
toc: |
  <li class="toc-item"><a href="#overview">Overview</a></li>
  <li class="toc-item"><a href="#objectives">Objectives</a></li>
  ... (one line per section)
---

{% include brd/cover.html
   title="[Title]"
   subtitle="[Subtitle]"
   id="[BRD-YYYY-NNN]"
   status="draft"
   version="0.1"
   author="[Author]"
   created="[YYYY-MM-DD]"
%}

{% capture section_content %}
<p>Content goes here...</p>
{% endcapture %}
{% include brd/section.html id="overview" num="1" title="Overview" content=section_content %}
```

### Available includes and their parameters

**`brd/cover.html`**
Parameters: `title`, `subtitle`, `id`, `status` (draft|review|approved), `version`, `author`,
`created`, `updated` (optional, defaults to created), `extra_meta` (optional raw HTML for
additional meta fields)

**`brd/section.html`**
Parameters: `id` (anchor id), `num` (section number), `title`, `content` (HTML string)
Content is passed via a `{% capture %}` block. Can contain any HTML — paragraphs, lists,
tables, nested headings (h3), callouts, timelines, or raw inline HTML for one-off needs.

**`brd/callout.html`**
Parameters: `type` (info|warning|success), `content` (HTML string)
Used inside a section's capture block.
Example: `{% include brd/callout.html type="warning" content="<strong>Note:</strong> ..." %}`

**`brd/timeline-item.html`**
Parameters: `state` (done|active|pending), `date`, `title`, `desc` (optional)
Used inside a `<ul class="timeline">` inside a section's capture block.

### Tables

Write tables directly as HTML inside capture blocks — no include needed:
```html
<table class="data-table">
  <thead><tr><th>ID</th><th>Requirement</th><th>Priority</th></tr></thead>
  <tbody>
    <tr>
      <td style="font-family:'JetBrains Mono',monospace;font-size:12px">FR-01</td>
      <td>Description</td>
      <td><span class="priority priority-high">High</span></td>
    </tr>
  </tbody>
</table>
```

Priority badge classes: `priority-high`, `priority-medium`, `priority-low`
Status badge classes: `badge-draft`, `badge-review`, `badge-approved`

### Rules
- Every section needs a unique `id` that matches a ToC entry in the front matter
- Section numbers must stay sequential
- Never add inline `<style>` blocks — all styles come from the layout
- Never add `<html>`, `<head>`, or `<body>` tags — Jekyll handles the page shell
- The `toc` front matter value must list every section in order

### Preview

After building the document, render a preview inline using the visualize `show_widget` tool.
To preview, fetch the CSS from `_includes/brd/shell-head.html` in the repo to get the correct
styles, then render the document's visible content (cover + sections) inside a scaled wrapper.
Loading messages should be short and calm: "Building your document...", "Updating the preview..."

After rendering, always ask the user if they want to save the document to the Tymraft site.
Use plain language: "Would you like to save this to the Tymraft site so the team can access it?"

Also offer:
- Making changes to a specific part of the document
- Adding or removing sections

Always ask about saving first — it is the primary action after a document is built or updated.

## Step 5 — Iterative editing

On every subsequent user message, apply their requested changes to the working document file,
then re-render the preview using show_widget. Always render the complete document — never
partial sections.

For small changes (a single field, one sentence), update only that part of the file content
in memory. There is no need to re-read the file from GitHub — keep the current state in context.

After every re-render, always ask again whether the user wants to save the updated version:
- "Want me to save this updated version to the site?"
- "Shall I save this to the Tymraft site so the team can see it?"

Editing rules (enforce strictly):
- Never add `<style>` blocks or inline CSS beyond what's already in the data file
- Never remove sections the user hasn't asked to remove
- Section numbers must stay sequential after additions or removals
- Keep the ToC front matter in sync with actual sections at all times
- Priority and status badge classes must match the allowed values above

## Step 6 — Saving to the site

When the user agrees to save, write the document to `docs/` in the Tymraft GitHub repo using
the github tool. Derive a simple, readable name from the document title (lowercase words
separated by hyphens, e.g. `my-project-brd.html`). Do not ask the user to provide or approve
a filename — just pick a sensible one and mention it after saving.

Write the file exactly as composed — the Jekyll front matter and include calls are the complete
file content. No wrapper HTML needed.

After saving, confirm in plain language:
- "Done! Your document is now live at: https://tymraft.github.io/docs/[name]"
- "The team can find it in the document library at tymraft.github.io"

## What to read next

- `references/type-picker.html` — the inline widget shown when no document type is specified
- `references/brd-template-guide.md` — section-by-section guide to what content goes in each BRD section
