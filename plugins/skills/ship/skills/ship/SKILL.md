---
name: ship
description: "Semver bump → commit → tag → push. Trigger: \"ship\", \"ship patch/minor/major\", \"release\", \"tag a release\"."
user_invocable: true
args: "[patch|minor|major|beta|<explicit-version>]"
---

# Ship Skill

Bump version, commit, tag, push. One plan → one confirm → auto-execute. Only pause on real conflicts.

## Step 1: Read Project Context

Read `CLAUDE.md`. Extract:
- Version file location and field name (e.g. `## Version:` in `.toc`, `"version"` in `package.json`)
- Release platform (CurseForge, GitHub Releases, npm, etc.)
- Branch and tag conventions
- Any pre-release steps required

## Step 2: Parse Arguments

- `patch` / `minor` / `major` — auto-calculate next version
- `beta` — create or increment a beta pre-release
- Explicit version like `0.2.0` or `1.0.0-beta`
- No argument — ask which bump type they want

## Step 3: Check Working Tree

Run `git status --porcelain`. If non-empty:

> **Conflict:** Working tree has uncommitted changes:
> `<list changed files>`
> Abort and commit/stash first, or continue (changes included in release commit)?

Do NOT continue until user decides.

## Step 4: Read Current Version and Existing Tags

Read version file (from CLAUDE.md), extract current version. Parse as semver: `MAJOR.MINOR.PATCH[-PRERELEASE]`

```bash
git tag --list 'v*' --sort=-version:refname
```

If calculated tag already exists:

> **Conflict:** Tag `v<new>` already exists. Force-retag, pick different version, or abort?

Do NOT continue until user decides.

### Bump rules

| Input | Behavior |
|-------|----------|
| `patch` | Strip pre-release if present (promote to stable). Otherwise increment PATCH. |
| `minor` | Increment MINOR, reset PATCH to 0, strip pre-release. |
| `major` | Increment MAJOR, reset MINOR and PATCH to 0, strip pre-release. |
| `beta` | Tag-only. Derive from existing `vX.Y.Z-beta*` tags. No file changes. |
| explicit | Use as-is. Validate semver format. |

### Beta bump rules

Look at existing tags matching `vX.Y.Z-beta*` (where X.Y.Z is current stable version):
- None exist → `X.Y.Z-beta`
- `vX.Y.Z-beta` exists → `X.Y.Z-beta-1`
- `vX.Y.Z-beta-N` is highest → `X.Y.Z-beta-(N+1)`

Beta bumps skip Steps 6, 7, 8 — tag only, then push.

## Step 5: Run Release Prep (if available)

Check if `release-prep:release-prep` agent is listed in system-reminder skills/agents. If available, spawn it as pre-flight check.

If it reports blockers:

> **Conflict:** Release prep found blockers:
> `<blocker list>`
> Continue anyway or abort?

Do NOT continue until user decides. If no blockers, proceed silently.

If not available, skip silently.

## Step 5.5: Sync README (if available)

Check if `update-readme` skill is listed in system-reminder skills. If available, run it automatically — no prompt. Include any changes in the release commit.

If not available, skip silently.

## Step 6: Show Plan and Get Single Confirmation

Generate the changelog entry (Step 7 logic below) but don't write it yet. Then show the full plan:

```
Ship Plan
─────────────────────────────────────
Current version:  <current>
New version:      <new>
Tag:              v<new>

Will do:
  1. Update version in <version-file>
  2. Append changelog entry (shown below)
  3. Commit: release: v<new-version>
  4. Tag: v<new-version>
  5. Push to origin/main

Changelog entry:
──────────────────
## [<new-version>] -- <YYYY-MM-DD>
<generated entries>
──────────────────

Proceed? (yes to run all steps)
```

Do NOT continue until confirmed. This is the ONE human checkpoint for the happy path.

## Step 7: Update Version File

Edit the version field in the file specified in CLAUDE.md.

## Step 8: Write CHANGELOG.md

If `CHANGELOG.md` exists and has no section for the new version:

1. Generate changelog from commits since last tag:
```bash
git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD --oneline --no-decorate
```

2. Group by conventional commit prefixes (feat→Added, fix→Fixed, refactor/chore→Changed). No prefix → Changed. Only include categories that have entries.

3. Prepend section to CHANGELOG.md. No user prompt — already shown and approved in Step 6.

## Step 9: Commit

Stage only changed files. Commit immediately — no prompt.

Commit message: `release: v<new-version>`

Co-author line: `Co-Authored-By: Claude <noreply@anthropic.com>`

## Step 10: Tag

```bash
git tag -a v<new-version> -m "Release v<new-version>"
```

No prompt — approved in Step 6.

## Step 11: Push

```bash
git push origin main --follow-tags
```

No prompt — approved in Step 6. If push fails, report exact error and let user decide next action.

## Step 12: Summary

```
Released v<new-version>
- Version file updated
- Commit: <short-hash>
- Tag: v<new-version>
- Pushed to origin/main
```

## Rules

- One confirmation only (Step 6). Never add extra prompts on the happy path.
- Pause ONLY on real conflicts: uncommitted changes, tag collision, release-prep blockers.
- Never force-push. Push fails → report error, ask user.
- Never amend commits. Always create new ones.
