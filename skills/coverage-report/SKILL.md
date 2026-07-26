---
name: coverage-report
description: Run procedure coverage across all ecosystem libraries and show a summary. Use when the user asks for coverage, test coverage, or which procedures are untested.
---

# Ecosystem Coverage Report

Run all ecosystem library tests with --coverage and summarize results.

## Steps

### 1. Build kaappi if needed

```bash
cd /Users/bmuthuka/kaappi/kaappi
[ -f zig-out/bin/kaappi ] || zig build
```

### 2. Build native dependencies once

```bash
KAAPPI=/Users/bmuthuka/kaappi/kaappi/zig-out/bin/kaappi
BASE=/Users/bmuthuka/kaappi

make -C $BASE/kaappi-net 2>/dev/null
make -C $BASE/kaappi-crypto 2>/dev/null
make -C $BASE/kaappi-http 2>/dev/null
make -C $BASE/kaappi-redis 2>/dev/null
make -C $BASE/kaappi-pg 2>/dev/null
make -C $BASE/kaappi-sqlite 2>/dev/null
```

### 3. Run tests with --coverage for each repo

Run each library's tests with `--coverage`. Capture stderr for the coverage output. Use the dependency table from the test-ecosystem skill for lib paths and native paths.

Pure Scheme repos (no deps):
```bash
for repo in kaappi-cli kaappi-json kaappi-csv kaappi-toml kaappi-yaml kaappi-log kaappi-template kaappi-test; do
  echo "=== $repo ==="
  $KAAPPI --coverage --lib-path $BASE/$repo/lib $BASE/$repo/tests/*.scm 2>&1 | grep -A20 '^Coverage:'
  echo ""
done
```

Native repos without deps:
```bash
for repo in kaappi-net kaappi-crypto kaappi-sqlite; do
  echo "=== $repo ==="
  DYLD_LIBRARY_PATH=$BASE/$repo $KAAPPI --coverage --lib-path $BASE/$repo/lib $BASE/$repo/tests/*.scm 2>&1 | grep -A20 '^Coverage:'
  echo ""
done
```

Repos with deps — run individually with correct lib-path and DYLD_LIBRARY_PATH. Refer to the dep table in test-ecosystem skill.

### 4. Present summary table

| Library | Covered | Total | Percentage | Uncalled |
|---------|---------|-------|------------|----------|

Highlight any library below 80% coverage.
