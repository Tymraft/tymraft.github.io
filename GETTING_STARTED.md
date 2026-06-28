# Getting Started

> This guide is read by Claude to self-install the Tymraft plugin environment.
> To install, say: "Read my getting started guide at https://tymraft.github.io/GETTING_STARTED.md and set me up."

---

## Instructions for Claude

When you read this file, follow these steps **in order without asking the user to do anything manually**:

---

### Step 1 — Install the plugin via the Claude marketplace

This is fully automated. You will guide the user through adding the Tymraft repository as a personal plugin marketplace.

**For claude.ai (web/desktop):**

Tell the user:
> "To install the Tymraft plugin, open the **Customize** menu in the left sidebar, go to the **Plugins** tab, click the **+** button in the Personal plugins section, select **Add marketplace**, then choose **Add from a repository** and enter: `Tymraft/tymraft.github.io`"

Wait for them to confirm it's done, then continue.

**For Claude Code (terminal):**

Tell the user to run these two commands inside Claude Code:

```
/plugin marketplace add Tymraft/tymraft.github.io
/plugin install tymraft@tymraft
```

Then run `/reload-plugins` to activate.

Once the plugin is installed via the marketplace, it will auto-load on every session — no manual setup needed in future conversations.

---

### Step 2 — Load the plugin manifest into this session

Fetch `plugin/.claude-plugin/plugin.json` from the `Tymraft/tymraft.github.io` GitHub repo. Load the `system_prompt` field and apply it as your operating context for this project.

---

### Step 3 — Load all skills

Fetch the `plugin/skills/` directory from the repo. For each subdirectory found:
- Read its `SKILL.md` file
- Register the skill name, description, and trigger conditions in your context
- Note any slash commands the skill exposes

If the skills directory is empty, note that no skills are installed yet and continue.

---

### Step 4 — Confirm setup

Respond with a short confirmation:
- Plugin version loaded
- Number of skills registered
- List of available slash commands (if any)
- One line confirming you will auto-trigger skills on every prompt

---

## Project Context

Tymraft is a team using Claude as an integrated assistant across development, planning, and operations workflows. The plugin lives at `https://tymraft.github.io` and is managed via the `Tymraft/tymraft.github.io` GitHub repository.

Claude has GitHub access and can read, create, and update files in the repo directly. Always commit to `main` unless told otherwise.

---

## Skills

Skills live in `plugin/skills/<skill-name>/SKILL.md`. Each skill file defines:
- What the skill does
- What triggers it (keywords, slash commands, intent patterns)
- Step-by-step instructions for Claude to follow when triggered

To add a new skill, create a folder under `plugin/skills/` and add a `SKILL.md`.

---

## MCP Servers

No MCP servers are configured yet. This section will be updated as integrations are added.
