---
name: update-readme
description: >
  Sync README.md with current project features by reading story docs and CLAUDE.md.
  Use when the user says "update readme", "sync readme", "readme is out of date",
  "/update-readme", or before shipping a release. Also use proactively after a new
  epic is completed or when multiple stories have been implemented since the last
  README update.
user_invocable: true
---

# Update README Skill

Keep README.md accurate by reading the source of truth (stories + CLAUDE.md) and rewriting only the sections that describe functionality.

## Step 1: Read Project Context

Read `CLAUDE.md` first. It contains:
- Where story/epic docs live
- Which README sections should be updated vs left alone
- Feature writing style guidance (if any)
- Project architecture and capability pillars

## Step 2: Read Sources

1. `CLAUDE.md` — architecture, key capabilities, design constraints
2. Story/epic docs (path from CLAUDE.md) — check which stories are **checked off** (`[x]`). Only completed stories count as shipped features.
3. `README.md` — current state, to avoid rewriting things already correct

## Step 3: Rewrite Sections

Update sections specified in CLAUDE.md's "README sections to update" list. If not specified, default to:

- **Intro paragraph** — key capability pillars
- **Features** — one bullet per distinct user-visible capability. Only completed stories. Short and scannable.
- **Usage** — slash commands or CLI flags, derived from source files

Leave alone sections specified in CLAUDE.md. If not specified, default to leaving alone: Installation, Requirements, Built With, License, Contributing.

## Step 4: Feature List Rules

Good feature bullets:
- User-visible, not implementation details
- Short (one clause)
- Ordered by user importance

Bad: "Lazily initializes arrays with deduplication"
Good: "**Rating History** — progression graph per character/bracket"

## Output

Rewrite README.md in place. No summary comment after — the diff speaks for itself.

If README is already accurate, say so and skip the write.
