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
claude plugin install wow-api-research@nelx-claude
```

---

## Available Plugins

| Name | Type | Tags | Description |
|------|------|------|-------------|
| `implement-story` | agent | — | Implements a feature story end-to-end: reads spec, writes code, runs tests |
| `lua-linter` | agent | wow | Lints and fixes Lua files, focused on WoW addon conventions |
| `wow-api-research` | skill | wow | Researches WoW API usage, finds correct event/function signatures |

Plugins tagged `wow` are WoW addon-specific. All others are general-purpose.

---

## Using Plugins

### Agents

Installed to `.claude/agents/`. Claude Code selects them automatically based on task context, or trigger explicitly:

> "Use the implement-story agent to build this feature."
> "Implement the feature described in STORY-42.md"

### Skills

Installed to `.claude/skills/`. Invoked via slash command or by Claude when task matches:

```
/wow-api-research
```

---

## Adding a Plugin

1. Create `plugins/<name>/` directory
2. Add `AGENT.md` or `SKILL.md` with prompt content
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
    marketplace.json    marketplace listing
    plugin.json         metadata
  plugins/
    implement-story/
      AGENT.md
      plugin.json
    lua-linter/
      AGENT.md
      plugin.json
    wow-api-research/
      SKILL.md
      plugin.json
  README.md
```
