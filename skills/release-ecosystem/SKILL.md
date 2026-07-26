---
name: release-ecosystem
description: Cut a GitHub release for a kaappi-* ecosystem library — runs tests, updates CHANGELOG.md, bumps version in kaappi.pkg, commits, tags, pushes, creates the GitHub release, and verifies CI on the release commit. Usage /release-ecosystem <repo> [version], e.g. /release-ecosystem kaappi-json 0.3.0. Use when asked to release, cut, tag, publish, or ship a version of an ecosystem library. Not for the core kaappi repo — that has its own /github-release skill.
---

# Release Ecosystem Library

Full release process for a `kaappi-*` ecosystem library: tests, changelog,
version bump, commit, annotated tag, push, GitHub release, CI verification.

The repo lives at `/Users/bmuthuka/kaappi/<repo>`. Run all git commands with
`git -C /Users/bmuthuka/kaappi/<repo>` and all gh commands with
`-R kaappi/<repo>` so the current working directory doesn't matter.

Why tag at all: unversioned `thottam install <repo>` tracks main HEAD, but
pinned `<repo>@X.Y.Z` resolves the git tag — the tag is what makes a version
installable.

## Arguments

1. `<repo>` (required) — e.g. `kaappi-json`
2. `[version]` (optional) — e.g. `0.3.0`, no `v` prefix. If omitted, see Step 2.

## Prerequisites

Check all of these before touching anything:

- `/Users/bmuthuka/kaappi/<repo>/kaappi.pkg` exists — otherwise this is not an
  ecosystem library (for the core interpreter, use the core repo's
  `/github-release` skill instead)
- Working tree clean: `git -C <dir> status --porcelain` is empty
- On `main` and in sync: `git -C <dir> fetch origin` then
  `git -C <dir> rev-parse main origin/main` — both SHAs equal, and
  `git -C <dir> branch --show-current` is `main`
- `gh auth status` succeeds
- CI green on main HEAD:
  `gh run list -R kaappi/<repo> --branch main --limit 1 --json conclusion,headSha`
  — conclusion `success`. No runs at all is fine for a brand-new repo; say so
  and continue.

If anything fails, stop and report instead of proceeding.

## Step 1: Determine current version

```bash
git -C <dir> tag -l 'v*' --sort=-v:refname | head -1
```

No tags means this is the first release. Cross-check the `version:` field in
`kaappi.pkg`: most repos don't have one until their first release (normal);
if it exists but disagrees with the latest tag, flag it before continuing.

## Step 2: Choose the new version

If the user gave a version, use it. Otherwise propose one from the changes
since the last tag — **patch** = fixes only, **minor** = new features,
**major** = breaking changes (pre-1.0, minor for features is fine; first
release of an existing lib is typically `0.1.0`) — and confirm with the user
before continuing.

## Step 3: Draft release notes

```bash
git -C <dir> log $(git -C <dir> tag -l 'v*' --sort=-v:refname | head -1)..HEAD --oneline --no-merges
```

(First release: `git -C <dir> log --oneline --no-merges` for the full history.)

- Primary source is the `[Unreleased]` section of `CHANGELOG.md` when one
  exists; the log fills in anything not yet recorded.
- Group entries Keep a Changelog style: `### Added` / `### Changed` /
  `### Fixed` / `### Removed`.
- Leave out infra-only commits (CI tweaks, Codecov config) — the changelog is
  for library consumers.

## Step 4: Update CHANGELOG.md

Keep a Changelog format, matching `kaappi-cli/CHANGELOG.md`. If the file is
missing, create it. After editing it must look like:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD

### Added
- ...

### Fixed
- ...

## [previous version] - previous date
...
```

Clear the `[Unreleased]` content but keep the heading; insert the new
`## [X.Y.Z] - YYYY-MM-DD` section (today's date) below it; preserve all older
sections.

## Step 5: Update version in kaappi.pkg

Set `version: X.Y.Z` (no `v` prefix). If the field doesn't exist yet, add it
on the line after `name:`.

## Step 6: Build and test locally

Run the repo's tests the same way `/test-ecosystem <repo>` does — see
`/Users/bmuthuka/kaappi/.claude/skills/test-ecosystem/SKILL.md` for the
per-category commands (pure Scheme / native `make` build / with
dependencies, including `--lib-path` and `DYLD_LIBRARY_PATH` construction).
Use `kaappi` from PATH, falling back to
`/Users/bmuthuka/kaappi/kaappi/zig-out/bin/kaappi`.

- If `kaappi.pkg` has `build: make`, run `make` first (and `make` each native
  dependency).
- `kaappi-pg` / `kaappi-redis` tests need a running PostgreSQL / Redis. If the
  service isn't available locally, run whatever suites don't need it, say
  which were skipped, and treat Step 10's CI verification as the gate.

Fix failures before proceeding — never release on a red suite.

## Step 7: Commit and tag

```bash
git -C <dir> add CHANGELOG.md kaappi.pkg
git -C <dir> commit -m "Release vX.Y.Z"
git -C <dir> tag -a vX.Y.Z -m "Release vX.Y.Z"
```

Subject only, no body. Annotated tag, matching existing releases.

## Step 8: Push

**STOP** and ask for confirmation first, unless the user already authorized
pushing in this conversation (e.g. the request said "release and push" or
listed pushing as a step). Pushing publishes the release.

```bash
git -C <dir> push origin main
git -C <dir> push origin vX.Y.Z
```

Push the one tag by name — never `--tags`.

## Step 9: Create the GitHub release

Write the confirmed notes to a temp file first — inline `--notes` breaks on
backticks and quotes. End the notes with a compare link:

```markdown
**Full Changelog**: https://github.com/kaappi/<repo>/compare/vPREV...vX.Y.Z
```

(First release: `https://github.com/kaappi/<repo>/commits/vX.Y.Z`.)

```bash
gh release create vX.Y.Z -R kaappi/<repo> --title "vX.Y.Z" --notes-file <file>
```

Backfilling a release for an *older* existing tag when a newer release
already exists: add `--latest=false` so the newest version keeps the
"Latest" badge.

## Step 10: Verify CI on the release commit

The push to main triggers CI (tag pushes trigger nothing). Find the run for
the release commit and watch it:

```bash
gh run list -R kaappi/<repo> --branch main --limit 3 --json databaseId,headSha,status,displayTitle
gh run watch <run-id> -R kaappi/<repo> --exit-status --interval 15
```

Ecosystem CI builds the interpreter from source, so expect ~3 minutes.
Report the release URL and the CI conclusion together — the release isn't
done until CI is green.

## Error recovery

**Before push** (undo commit and tag, fix, redo):

```bash
git -C <dir> tag -d vX.Y.Z
git -C <dir> reset --soft HEAD~1
```

**After push:**

```bash
gh release delete vX.Y.Z -R kaappi/<repo> --yes   # keeps the tag
git -C <dir> push origin --delete vX.Y.Z          # removes the remote tag
git -C <dir> tag -d vX.Y.Z
```

Don't rewrite pushed history on main — fix forward with a new commit (and a
new patch release if the bad state was tagged).
