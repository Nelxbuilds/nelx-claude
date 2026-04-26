---
name: bug
description: "Track bugs in docs/bugs.md. Add, list, or close entries. Trigger: \"log bug\", \"add bug\", \"list bugs\", \"close bug #N\"."
user_invocable: true
args: "[add <description> [file:line] | list]"
---

# Bug Skill

Manage `docs/bugs.md` — the single source of truth for open bugs. Never add bugs to story/epic docs.

## File Format

`docs/bugs.md` must contain this table (create if missing):

```markdown
# Bugs

| # | Description | File | Status |
|---|-------------|------|--------|
```

Rows:
```
| 1 | Short description | `File.lua:42` | open |
| 2 | Another bug | — | resolved |
```

- `File` column: `\`File.lua:42\`` if known, `—` if not
- `Status`: `open` or `resolved`

## Commands

### add

Triggers: "log bug", "add bug", "found a bug", "bug: ...", `/bug add ...`

Steps:
1. Read `docs/bugs.md`. If missing, create with header + empty table.
2. Find highest existing `#`. New ID = highest + 1 (start at 1 if empty).
3. Extract description from user input. If file:line mentioned, put in File column.
4. Append row. Status = `open`.
5. Confirm: `Bug #N logged.`

### list

Triggers: "list bugs", "show bugs", "open bugs", "what bugs are open", `/bug list`

Steps:
1. Read `docs/bugs.md`.
2. Print only `open` rows. If none: `No open bugs.`

## Rules

- If user gives no command and just mentions a bug description, treat as `add`.
- Keep descriptions short (one line, no periods).
- Don't invent file:line — only record if user provides it or it's evident from context.
- When fixing a bug, update status to `resolved` in bugs.md as part of the fix — no separate close command needed.
