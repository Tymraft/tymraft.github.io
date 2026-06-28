---
name: jira
description: >
  Trigger this skill on ANY of the following — when in doubt, trigger:

  EXPLICIT triggers:
  - "/tymraft:jira" anywhere in the prompt (primary invocation)
  - "/jira" anywhere in the prompt
  - "create a jira ticket", "create a ticket", "new ticket", "log a ticket"
  - "update ticket", "edit ticket", "modify ticket", "change ticket"
  - "delete ticket", "close ticket", "remove ticket"
  - "read ticket", "show ticket", "get ticket", "look up ticket"
  - "transition ticket", "move ticket to", "set status of"
  - "what's in TRD", "look up TRD", "show me TRD"

  IMPLICIT triggers — ANY of these patterns must trigger this skill:
  - Any mention of "TRD" (with or without a number, e.g. "TRD-48", "TRD-NEW", "TRD ticket")
  - Any Jira ticket key matching TRD-NNN
  - "add a story for...", "we need a task for...", "file a bug for..."
  - "mark TRD-NNN as done", "assign TRD-NNN to...", "what's the status of TRD-NNN"
  - "ticket", "story", "task", "bug", "epic" used in a project/work-tracking context
  - Any request to view, read, open, check, update, or create a work item

  Space: always TRD. Never ask the user which space.

  When in doubt, trigger.
---

# /jira Skill — TRD Space

This skill manages Jira tickets in the **TRD** project space using the connected Atlassian Rovo MCP.

All actions follow this sequence:
1. Get cloudId (see **MCP Startup** below — do this first, always)
2. Gather required information (ask once if anything critical is missing)
3. **Render the Jira ticket preview widget** (baked HTML below)
4. Confirm with the user (mutating actions only)
5. Execute the MCP action
6. Report the result with the ticket key and URL

---

## MCP Startup — ALWAYS DO THIS FIRST

Before any Jira tool call, Claude MUST call `getAccessibleAtlassianResources` to obtain the `cloudId`.

- **Never** assume the connection is broken just because a prior `search` call failed — `search` uses different auth than `getJiraIssue`. Always attempt `getAccessibleAtlassianResources` first.
- Cache the `cloudId` for the session once obtained. It will be the `id` field from the first result (e.g. `0dd1fd2a-450a-4cb9-a798-c14a4e1e6612`).
- If `getAccessibleAtlassianResources` itself fails, then report the connection issue to the user.
- Do not use `Atlassian Rovo:search` as a proxy for Jira issue lookup — use `getJiraIssue` or `searchJiraIssuesUsingJql` directly with the cloudId.

---

## MCP Tool Usage

Always use the connected Atlassian Rovo MCP tools:
- **Read**: `getJiraIssue` or `searchJiraIssuesUsingJql`
- **Create**: `createJiraIssue` — always set `cloudId` from `getAccessibleAtlassianResources`
- **Update**: `editJiraIssue`
- **Transition/Delete**: `transitionJiraIssue` or `editJiraIssue` with resolution

Always use `projectKey: "TRD"`.

---

## Preview Widget

Before **every** create, update, or delete action, call `show_widget` with the HTML below, populated with the ticket's data. Also render for READ actions (no confirm step needed for reads).

- **Create / Update**: render the ticket card in its final state (what it will look like after the action).
- **Delete**: render the card with `data-deleted="true"` — this shows it collapsed and struck through.
- **Multiple tickets**: stack all affected cards inside `#jira-stack` — each is independently collapsible.

### Loading messages
- Create: `["Drafting ticket...", "Rendering preview..."]`
- Update: `["Loading changes...", "Rendering preview..."]`
- Delete: `["Marking for deletion..."]`
- Read: `["Fetching ticket..."]`

### Widget HTML

Inject ticket data by replacing the placeholder values. All fields are plain text unless noted. Render one `.jira-card` per ticket inside `#jira-stack`.

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  *{box-sizing:border-box;margin:0;padding:0;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif}
  body{background:#f4f5f7;padding:16px;min-height:100vh}
  #jira-stack{display:flex;flex-direction:column;gap:10px;max-width:680px;margin:0 auto}
  .jira-card{background:#fff;border-radius:4px;box-shadow:0 1px 3px rgba(0,0,0,.12);overflow:hidden;transition:all .2s}
  .jira-card[data-deleted="true"]{opacity:.45}
  .card-header{display:flex;align-items:center;gap:8px;padding:10px 14px;cursor:pointer;user-select:none;border-bottom:1px solid #ebecf0}
  .card-header:hover{background:#f4f5f7}
  .issue-type{width:16px;height:16px;border-radius:2px;flex-shrink:0;display:flex;align-items:center;justify-content:center;font-size:10px;font-weight:700;color:#fff}
  .type-story{background:#63ba3c}
  .type-task{background:#4bade8}
  .type-bug{background:#e5493a}
  .type-epic{background:#904ee2}
  .type-subtask{background:#4bade8}
  .issue-key{font-size:12px;font-weight:600;color:#5e6c84;flex-shrink:0;min-width:72px}
  .jira-card[data-deleted="true"] .issue-key{text-decoration:line-through}
  .issue-summary{font-size:13px;font-weight:500;color:#172b4d;flex:1;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
  .jira-card[data-deleted="true"] .issue-summary{text-decoration:line-through;color:#8993a4}
  .status-badge{font-size:10px;font-weight:700;padding:2px 7px;border-radius:3px;text-transform:uppercase;letter-spacing:.04em;flex-shrink:0}
  .s-todo{background:#dfe1e6;color:#42526e}
  .s-progress{background:#deebff;color:#0052cc}
  .s-review{background:#e3fcef;color:#006644}
  .s-done{background:#e3fcef;color:#006644}
  .s-blocked{background:#ffebe6;color:#bf2600}
  .chevron{margin-left:auto;font-size:11px;color:#97a0af;transition:transform .2s;flex-shrink:0}
  .card-body{padding:14px;display:grid;grid-template-columns:1fr 1fr;gap:10px;border-top:1px solid #ebecf0}
  .card-body.hidden{display:none}
  .field{display:flex;flex-direction:column;gap:2px}
  .field-label{font-size:10px;font-weight:600;text-transform:uppercase;letter-spacing:.05em;color:#5e6c84}
  .field-value{font-size:12px;color:#172b4d}
  .field-value.muted{color:#8993a4;font-style:italic}
  .description-field{grid-column:1/-1}
  .desc-text{font-size:12px;color:#172b4d;line-height:1.6;white-space:pre-wrap;max-height:160px;overflow-y:auto;border:1px solid #ebecf0;border-radius:3px;padding:8px;background:#fafbfc}
  .priority-icon{display:inline-block;width:10px;height:10px;border-radius:50%;margin-right:4px;vertical-align:middle}
  .p-highest,.p-critical{background:#cd1316}
  .p-high{background:#e97f33}
  .p-medium{background:#f5cd47}
  .p-low{background:#2d8738}
  .p-lowest{background:#57a55a}
  .deleted-banner{font-size:11px;font-weight:600;color:#bf2600;text-transform:uppercase;letter-spacing:.05em;padding:4px 14px;background:#ffebe6;grid-column:1/-1}
</style>
</head>
<body>
<div id="jira-stack">

  <!-- Repeat .jira-card for each ticket -->
  <div class="jira-card" data-deleted="false">
    <div class="card-header" onclick="toggle(this)">
      <!-- Issue type badge: add class type-story / type-task / type-bug / type-epic / type-subtask -->
      <span class="issue-type type-story" title="Story">S</span>
      <span class="issue-key">TRD-???</span>
      <span class="issue-summary">Ticket summary goes here</span>
      <!-- Status: class s-todo / s-progress / s-review / s-done / s-blocked -->
      <span class="status-badge s-todo">To Do</span>
      <span class="chevron">▼</span>
    </div>
    <div class="card-body">
      <!-- Show deleted-banner only when data-deleted="true" -->
      <!-- <div class="deleted-banner">⚠ This ticket will be deleted</div> -->

      <div class="field">
        <span class="field-label">Assignee</span>
        <span class="field-value muted">Unassigned</span>
      </div>
      <div class="field">
        <span class="field-label">Priority</span>
        <span class="field-value">
          <span class="priority-icon p-medium"></span>Medium
        </span>
      </div>
      <div class="field">
        <span class="field-label">Reporter</span>
        <span class="field-value">Jane Smith</span>
      </div>
      <div class="field">
        <span class="field-label">Labels</span>
        <span class="field-value muted">None</span>
      </div>
      <div class="field description-field">
        <span class="field-label">Description</span>
        <div class="desc-text">Full description text here...</div>
      </div>
    </div>
  </div>
  <!-- /end .jira-card -->

</div>
<script>
function toggle(header){
  var body=header.nextElementSibling;
  var chev=header.querySelector('.chevron');
  body.classList.toggle('hidden');
  chev.style.transform=body.classList.contains('hidden')?'rotate(-90deg)':'';
}
// Start all cards expanded
document.querySelectorAll('.card-body').forEach(function(b){b.classList.remove('hidden')});
</script>
</body>
</html>
```

---

## Field Reference

| Field | Notes |
|---|---|
| `data-deleted` | `"true"` for delete previews, `"false"` otherwise |
| Issue type class | `type-story` `type-task` `type-bug` `type-epic` `type-subtask` |
| Issue type letter | S / T / B / E / ↳ |
| Status class | `s-todo` `s-progress` `s-review` `s-done` `s-blocked` |
| Priority class | `p-highest` `p-critical` `p-high` `p-medium` `p-low` `p-lowest` |
| `TRD-???` | Use real key after creation; use `TRD-NEW` as placeholder before |

---

## Behaviour by Action

### CREATE
1. Extract: summary, type (default: Task), priority (default: Medium), description, assignee, labels
2. If summary is missing, ask for it. Infer everything else from context.
3. Render preview widget with `TRD-NEW` as the key, `data-deleted="false"`, status `s-todo`
4. Say: *"Here's what the ticket will look like — shall I create it?"*
5. On confirm → call `createJiraIssue` with `projectKey: "TRD"`
6. Report: *"Created [TRD-NNN](url)"*

### READ
1. Call `getJiraIssue` or `searchJiraIssuesUsingJql`
2. Render preview widget with real data (no confirm step needed)
3. Summarise key fields in one line below the widget

### UPDATE
1. Identify ticket key(s) and fields to change
2. Fetch current state via `getJiraIssue`
3. Render preview widget showing the ticket **after** the proposed change
4. Say: *"Here's how TRD-NNN will look after the update — shall I apply it?"*
5. On confirm → call `editJiraIssue`

### DELETE / TRANSITION TO DONE
1. Identify ticket key(s)
2. Fetch current state
3. Render preview widget with `data-deleted="true"` and show `.deleted-banner`
4. Say: *"TRD-NNN will be marked as deleted/closed — confirm?"*
5. On confirm → transition or resolve via MCP

### MULTIPLE TICKETS
- Stack all `.jira-card` elements inside `#jira-stack`
- All start expanded; user can collapse individually
- Confirm once for all: *"The above N tickets will be [action] — confirm?"*

---

## Rules
- **Always** call `getAccessibleAtlassianResources` first to get cloudId — even if you think it's cached. Never report a connection failure without trying this first.
- Do not use `Atlassian Rovo:search` to fetch Jira issues — it uses different auth. Use `getJiraIssue` or `searchJiraIssuesUsingJql` with the cloudId directly.
- Always use `projectKey: "TRD"` — never ask the user which project
- Never skip the preview widget — it must render before every mutating action, and also for reads
- Never fabricate ticket keys — use `TRD-NEW` until the real key is returned
- Keep widget HTML minimal — no external dependencies, no images
- If the MCP returns an error, show it plainly and suggest a fix
