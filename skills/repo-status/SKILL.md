---
name: repo-status
description: Show status across all kaappi repos — git branch, uncommitted changes, CI status, and how far ahead/behind origin. Use when the user asks for status, what's going on, what needs attention, or before starting a work session.
---

# Repo Status

Show a quick dashboard of all repos in the workspace.

## Steps

1. Run this to get git status for every repo:

```bash
for repo in kaappi kaappi-cli kaappi-json kaappi-csv kaappi-toml kaappi-yaml kaappi-log kaappi-template kaappi-test kaappi-net kaappi-crypto kaappi-http kaappi-email kaappi-pg kaappi-redis kaappi-sqlite kaappi-web kaappi-examples kaappi.github.io vscode-kaappi infra .github ci-images wiki; do
  dir="/Users/bmuthuka/kaappi/$repo"
  [ -d "$dir/.git" ] || continue
  cd "$dir"
  branch=$(git branch --show-current 2>/dev/null)
  dirty=$(git status --porcelain 2>/dev/null | wc -l | tr -d ' ')
  ahead=$(git rev-list --count origin/main..HEAD 2>/dev/null || echo "?")
  behind=$(git rev-list --count HEAD..origin/main 2>/dev/null || echo "?")
  echo "$repo|$branch|$dirty|$ahead|$behind"
done
```

2. Run this to get latest CI status for ecosystem repos:

```bash
for repo in kaappi kaappi-cli kaappi-json kaappi-csv kaappi-toml kaappi-yaml kaappi-log kaappi-template kaappi-test kaappi-net kaappi-crypto kaappi-http kaappi-email kaappi-pg kaappi-redis kaappi-sqlite kaappi-web kaappi-examples; do
  result=$(gh run list -R kaappi/$repo --limit 1 --json conclusion,event,createdAt --jq '.[0] | "\(.conclusion)\t\(.event)\t\(.createdAt[0:10])"' 2>/dev/null)
  echo "$repo: $result"
done
```

3. Present as a table:

| Repo | Branch | Dirty | Ahead | Behind | CI | Last Run |
|------|--------|-------|-------|--------|-----|----------|

Flag anything that needs attention: dirty working trees, repos ahead of origin (unpushed), repos behind origin (need pull), or CI failures.
