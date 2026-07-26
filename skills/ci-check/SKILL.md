---
name: ci-check
description: Check CI status and recent failures across all kaappi repos. Use when the user asks about CI, build status, what's broken, or what's failing.
---

# CI Check

Check GitHub Actions status across all repos.

## Steps

1. Get CI status for all repos with workflows:

```bash
for repo in kaappi kaappi-cli kaappi-json kaappi-csv kaappi-toml kaappi-yaml kaappi-log kaappi-template kaappi-test kaappi-net kaappi-crypto kaappi-http kaappi-email kaappi-pg kaappi-redis kaappi-sqlite kaappi-web kaappi-examples kaappi.github.io vscode-kaappi; do
  gh run list -R kaappi/$repo --limit 1 --json name,conclusion,event,createdAt,headBranch --jq '.[0] | "\(.conclusion)\t\(.event)\t\(.createdAt[0:10])\t\(.name)"' 2>/dev/null | while read line; do echo "$repo: $line"; done
done
```

2. Also check the nightly workflow:

```bash
gh run list -R kaappi/.github --workflow=nightly.yml --limit 3 --json conclusion,createdAt --jq '.[] | "\(.conclusion)\t\(.createdAt[0:10])"'
```

3. Present results as a table. For any failures, offer to show the failure logs:

```bash
gh run view <run-id> -R kaappi/<repo> --log-failed
```

4. Flag: repos with no CI runs, repos with failures, nightly failures, and repos where last run is more than 7 days old (stale).
