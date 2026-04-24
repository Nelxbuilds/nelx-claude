---
name: ship
description: "Semver bump → commit → tag → push. Trigger: \"ship\", \"ship patch/minor/major\", \"release\", \"tag a release\"."
user_invocable: true
args: "[patch|minor|major|beta|<explicit-version>]"
---

# Ship Skill

You help the user bump the project version, create a git tag, and push. You MUST ask for confirmation before every destructive or shared-state action.

## Step 1: Read Project Context

Read `CLAUDE.md` first. It contains:
- Version file location and field name (e.g. `## Version:` in a `.toc`, `"version"` in `package.json`)
- Release platform (CurseForge, GitHub Releases, npm, etc.)
- Branch and tag conventions
- Any pre-release steps required

## Step 2: Parse Arguments

The user provides one of:
- `patch` / `minor` / `major` — auto-calculate next version
- `beta` — create or increment a beta pre-release
- An explicit version like `0.2.0` or `1.0.0-beta`
- No argument — ask which bump type they want

## Step 3: Check Working Tree

Run `git status --porcelain`. If output is non-empty, warn:

```
⚠ Working tree has uncommitted changes:
<list changed files>
```

Ask: **"Abort and commit/stash first, or continue (changes will be included in release commit)?"**

Do NOT continue until user decides.

## Step 4: Read Current Version and Existing Tags

Read the version file (from CLAUDE.md) and extract the current version. Parse as semver: `MAJOR.MINOR.PATCH[-PRERELEASE]`

Read existing tags:
```bash
git tag --list 'v*' --sort=-version:refname
```

Use tags to detect if calculated version already exists (abort with warning if so).

### Bump rules

| Input | Behavior |
|-------|----------|
| `patch` | If pre-release suffix, strip it (promote to stable). Otherwise increment PATCH. |
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

## Step 5: Confirm

For stable bumps show:
```
Current version: <current>
New version:     <new>
Tag:             v<new>
```

For beta bumps show:
```
New tag:         v<new>
```

If tag already exists, warn and ask how to proceed. Ask: **"Proceed?"** Do NOT continue until confirmed.

## Step 6: Run Release Prep (if available)

Check if `release-prep` agent is available (listed in system-reminder skills/agents). If available, spawn it as a pre-flight check. If it reports blockers, show the report and ask whether to continue or abort.

If not available, skip this step silently.

## Step 6.5: Sync README (if available)

Check if `/update-readme` skill is available (listed in system-reminder skills). If available, run it automatically — no user prompt. Include any changes in the Step 8 commit.

If not available, skip this step silently.

## Step 7: Update Version File

Edit the version field in the file specified in CLAUDE.md.

## Step 8: Update CHANGELOG.md

If `CHANGELOG.md` exists and has no section for the new version:

1. Generate changelog from commits since last tag:
```bash
git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD --oneline --no-decorate
```

2. Group commits into categories based on conventional commit prefixes (feat→Added, fix→Fixed, refactor/chore→Changed, etc.). Commits without prefixes go under Changed.

3. Add section at top of CHANGELOG.md:
```markdown
## [<new-version>] -- <YYYY-MM-DD>

### Added
- <feat commits>

### Fixed
- <fix commits>

### Changed
- <other commits>
```

Only include categories that have entries. Show the generated entry and ask: **"Edit changelog before continuing?"**

## Step 9: Commit

Stage only changed files. Ask: **"Ready to commit?"**

Commit message: `release: v<new-version>`

## Step 10: Tag

Ask: **"Create tag v<new-version>?"**

```bash
git tag -a v<new-version> -m "Release v<new-version>"
```

## Step 11: Push

Ask: **"Push commit and tag to origin?"**

```bash
git push origin main --follow-tags
```

## Step 12: Summary

```
Done! Released v<new-version>
- Version file updated
- Commit: <short-hash>
- Tag: v<new-version>
- Pushed to origin/main
```

## Rules

- Always ask before commits, tags, pushes. Never auto-proceed.
- Never force-push. Push fails → tell user, let them decide.
- Never amend commits. Always create new ones.
- Co-author line on commit: `Co-Authored-By: Claude <noreply@anthropic.com>`
