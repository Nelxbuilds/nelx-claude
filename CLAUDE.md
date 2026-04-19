# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Personal Claude Code plugin marketplace — WoW addon development tools and general-purpose plugins. Published via `claude plugin marketplace add Nelxbuilds/nelx-claude`.

## Structure

```
.claude-plugin/
  plugin.json         root marketplace manifest
  marketplace.json    marketplace listing (used by claude plugin marketplace add)
plugins/
  agents/<name>/
    .claude-plugin/
      plugin.json     per-plugin manifest (required)
    agents/
      <name>.md       agent definition (frontmatter + prompt)
  skills/<name>/
    .claude-plugin/
      plugin.json     per-plugin manifest (required)
    skills/
      <name>/
        SKILL.md      skill definition (frontmatter + prompt)
    references/       optional reference files (wow-ui-designer only)
```

## Plugin Anatomy

Root `.claude-plugin/marketplace.json` lists all plugins for distribution:
```json
{
  "name": "nelx-claude",
  "owner": { "name": "Nelxbuilds" },
  "plugins": [
    { "name": "ship", "source": "./plugins/skills/ship", ... },
    { "name": "implement-story", "source": "./plugins/agents/implement-story", ... }
  ]
}
```

Each plugin directory MUST have `.claude-plugin/plugin.json` (required by Claude Code for discovery):
```json
{
  "name": "plugin-name",
  "description": "Description used for auto-triggering.",
  "author": { "name": "Nelxbuilds", "url": "https://github.com/Nelxbuilds" }
}
```

**Agents** — `agents/<name>.md` with frontmatter:
```markdown
---
name: plugin-name
description: Trigger description for Claude to auto-select this plugin.
tools: Read, Write, Edit, Glob, Grep, Bash
---
# Agent Title
...prompt body...
```

**Skills** — `skills/<name>/SKILL.md` with frontmatter:
```markdown
---
name: plugin-name
description: Trigger description. Add "Use when user says..." for slash-command triggers.
---
# Skill Title
...prompt body...
```

After creating a plugin: add its path to `plugins` array in `.claude-plugin/marketplace.json`.

## Plugin Design Rules

All agents and skills assume the **consuming project** has a `CLAUDE.md` that defines:
- File manifest location (e.g. `.toc` for WoW addons, `package.json` for others)
- Namespace/module conventions
- Story/epic doc paths
- Project-specific globals (e.g. SavedVariables)
- Design system, version file, release platform

Every agent starts with "Read `CLAUDE.md` first" — this is intentional. Plugins are context-agnostic by design; the consuming project's `CLAUDE.md` provides all project-specific configuration.

## Existing Plugins

| Name | Type | Purpose |
|------|------|---------|
| `implement-story` | agent | Reads story spec → writes code → updates file manifest → ticks AC checkboxes |
| `lua-linter` | agent | WoW addon static analysis: critical/perf/quality checks, does NOT fix code |
| `release-prep` | agent | Fixes manifest, creates `.pkgmeta`/`CHANGELOG`/`LICENSE`, runs linter, reports go/no-go |
| `review-addon` | agent | Compares plan (story docs) vs code reality, updates story doc checkboxes |
| `write-story` | agent | Interrogates requirements → writes unambiguous story doc, does NOT write code |
| `ship` | skill | Semver bump → release-prep → update-readme → commit → tag → push |
| `update-readme` | skill | Rewrites README from completed story docs + CLAUDE.md |
| `wow-api-research` | skill | Researches WoW API with sourced facts (Wowpedia, addon source, GitHub) |
| `wow-ui-designer` | skill | Writes production Lua UI code; never pseudocode; reads design system from CLAUDE.md |

## WoW-Specific Context

Target platform: **WoW Midnight 12.x**.

Key API rules embedded in plugins (do not regress):
- `ScrollBox`/`ScrollUtil` replaces `FauxScrollFrame`
- `MenuUtil.CreateContextMenu()` replaces `EasyMenu`/`UIDropDownMenuTemplate` (removed in 12.x)
- `BackdropTemplate` must be passed as 4th arg to `CreateFrame` — not callable on raw frames
- `SetGradient` uses `CreateColor` objects in 12.x
- SavedVariables must only be accessed inside `ADDON_LOADED` handler, never at file scope
- Timer polling: use `C_Timer.After`/`C_Timer.NewTicker`, not `OnUpdate` without elapsed guard

Reference files for `wow-ui-designer`:
- `plugins/skills/wow-ui-designer/references/midnight-ui-api.md` — frame/texture/scroll/animation API
- `plugins/skills/wow-ui-designer/references/settings-widgets.md` — slider, checkbox, dropdown, input patterns
