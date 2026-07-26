---
name: release-ecosystem
description: Cut a release for an ecosystem library — bump version in kaappi.pkg, update CHANGELOG.md, commit, tag, push, and create GitHub release. Usage /release-ecosystem kaappi-json 0.3.0
---

# Release Ecosystem Library

Cut a release for a kaappi ecosystem library.

## Arguments

- First arg: repo name (e.g. `kaappi-json`)
- Second arg: version number (e.g. `0.3.0`)

## Steps

### 1. Validate

- Confirm the repo exists at `/Users/bmuthuka/kaappi/<repo>`
- Confirm CI is passing: `gh run list -R kaappi/<repo> --limit 1 --json conclusion --jq '.[0].conclusion'`
- Confirm working tree is clean: `git -C /Users/bmuthuka/kaappi/<repo> status --porcelain`

### 2. Bump version

Update `kaappi.pkg`:
```
version: <new-version>
```

### 3. Update CHANGELOG.md

If CHANGELOG.md exists, add an entry at the top. If it doesn't exist, create one. Format:

```markdown
# Changelog

## <version> - <YYYY-MM-DD>

- <summary of changes since last tag>
```

Use `git log --oneline <last-tag>..HEAD` to summarize changes.

### 4. Commit, tag, push

```bash
cd /Users/bmuthuka/kaappi/<repo>
git add kaappi.pkg CHANGELOG.md
git commit -m "Release v<version>"
git tag v<version>
git push origin main --tags
```

### 5. Create GitHub release

```bash
gh release create v<version> -R kaappi/<repo> --title "v<version>" --notes "<changelog entry>"
```

### 6. Verify

Confirm the release appears: `gh release view v<version> -R kaappi/<repo>`
