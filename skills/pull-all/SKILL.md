---
name: pull-all
description: Pull latest from origin for all kaappi repos. Use when the user says pull all, sync, update repos, or wants to start fresh.
---

# Pull All Repos

Fetch and rebase all repos on origin/main.

## Steps

1. Run this:

```bash
for repo in kaappi kaappi-cli kaappi-json kaappi-csv kaappi-toml kaappi-yaml kaappi-log kaappi-template kaappi-test kaappi-net kaappi-crypto kaappi-http kaappi-email kaappi-pg kaappi-redis kaappi-sqlite kaappi-web kaappi-examples kaappi.github.io vscode-kaappi infra .github ci-images wiki; do
  dir="/Users/bmuthuka/kaappi/$repo"
  [ -d "$dir/.git" ] || continue
  echo "=== $repo ==="
  git -C "$dir" pull --rebase origin main 2>&1 | tail -1
done
```

2. Report which repos updated and which had issues (dirty working tree, conflicts, etc.)
