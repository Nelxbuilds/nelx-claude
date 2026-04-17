# nelx-claude

Personal plugin marketplace for Claude Code — agents and skills for WoW addon development and general use.

---

## Install

Requires [Claude Code](https://claude.ai/code).

```bash
claude plugin marketplace add Nelxbuilds/nelx-claude
```

Then install individual plugins:

```bash
claude plugin install implement-story@nelx-claude
claude plugin install lua-linter@nelx-claude
claude plugin install release-prep@nelx-claude
claude plugin install review-addon@nelx-claude
claude plugin install write-story@nelx-claude
claude plugin install ship@nelx-claude
claude plugin install update-readme@nelx-claude
claude plugin install wow-api-research@nelx-claude
claude plugin install wow-ui-designer@nelx-claude
```

---

## Available Plugins

### Agents

Installed to `.claude/agents/`. Claude Code selects them automatically based on task context, or trigger explicitly.

| Name | Tags | Description |
|------|------|-------------|
| `implement-story` | wow | Reads a story spec, writes production code, updates the file manifest, ticks acceptance criteria |
| `lua-linter` | wow | Static analysis for WoW addon Lua — critical bugs, performance issues, code quality. Read-only. |
| `release-prep` | wow | Fixes manifest fields, creates packaging files, runs linter, checks story completeness, go/no-go report |
| `review-addon` | wow | Compares story doc acceptance criteria against actual code; updates docs to reflect reality |
| `write-story` | wow | Interrogates requirements until unambiguous, then writes a tight story doc. Does not write code. |

### Skills

Installed to `.claude/skills/`. Invoked via slash command or by Claude when task matches.

| Name | Tags | Description |
|------|------|-------------|
| `ship` | wow | Semver bump → release-prep → README sync → commit → tag → push |
| `update-readme` | wow | Rewrites README from completed story docs and CLAUDE.md |
| `wow-api-research` | wow | Researches WoW API questions with sourced facts before they become story criteria or code |
| `wow-ui-designer` | wow | Writes production-ready WoW 12.x Lua UI code using the project's design system |

Plugins tagged `wow` are WoW addon-specific.

---

## Using Plugins

### Agents

Trigger by context or explicitly:

> "Implement story 3-2."
> "Use the write-story agent to plan this feature."
> "Is the addon ready to ship?"

### Skills

```
/ship patch
/wow-api-research what event fires when a rated match ends?
/wow-ui-designer
/update-readme
```

---

## Adding a Plugin

1. Create `plugins/agents/<name>/` or `plugins/skills/<name>/`
2. Add `AGENT.md` or `SKILL.md` with frontmatter + prompt body:

```markdown
---
name: my-plugin
description: Trigger description for Claude to auto-select this plugin.
tools: Read, Write, Edit, Glob, Grep, Bash
---
# My Plugin
...prompt body...
```

3. Add `plugin.json`:

```json
{
  "name": "my-plugin",
  "type": "agent",
  "version": "1.0.0",
  "description": "What it does in one line",
  "file": "AGENT.md"
}
```

4. Register in `.claude-plugin/marketplace.json` under `plugins[]`
5. Commit and push — available immediately

---

## Repo Structure

```
nelx-claude/
  .claude-plugin/
    marketplace.json    canonical plugin registry
    plugin.json         marketplace metadata
  plugins/
    agents/
      implement-story/
      lua-linter/
      release-prep/
      review-addon/
      write-story/
    skills/
      ship/
      update-readme/
      wow-api-research/
      wow-ui-designer/
        references/
          midnight-ui-api.md
          settings-widgets.md
  CLAUDE.md
  README.md
```
