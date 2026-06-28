# Getting Started

> This guide is read by Claude to self-install the Tymraft plugin environment.
> To install, say: "Read my getting started guide at https://tymraft.github.io/GETTING_STARTED.md and set me up."

---

## Instructions for Claude

When you read this file, follow these steps in order:

### 1. Read the plugin manifest

Fetch `plugin/.claude-plugin/plugin.json` from the `Tymraft/tymraft.github.io` GitHub repo. Load the `system_prompt` field and apply it as your operating context for this project.

### 2. Load all skills

Fetch the `plugin/skills/` directory from the repo. For each subdirectory found:
- Read its `SKILL.md` file
- Register the skill name, description, and trigger conditions in your context
- Note any slash commands the skill exposes

If the skills directory is empty, note that no skills are installed yet and continue.

### 3. Confirm setup

After completing steps 1–2, respond with a short confirmation:
- Plugin version loaded
- Number of skills registered
- List of available slash commands (if any)
- One line confirming you will auto-trigger skills on every prompt

---

## Project Context

Tymraft is a team using Claude as an integrated assistant across development, planning, and operations workflows. The plugin lives at `https://tymraft.github.io` and is managed via the `Tymraft/tymraft.github.io` GitHub repository.

Claude has GitHub access and can read, create, and update files in the repo directly.

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
