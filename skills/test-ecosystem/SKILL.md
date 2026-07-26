---
name: test-ecosystem
description: Run tests locally for one or all ecosystem libraries against the local kaappi build. Pass a repo name to test one (e.g. /test-ecosystem kaappi-json), or no args to test all. Use when the user asks to test a library, run ecosystem tests, or verify things work locally.
---

# Test Ecosystem Libraries

Run ecosystem library tests using the locally built kaappi interpreter.

## Steps

### 1. Build kaappi if needed

```bash
cd /Users/bmuthuka/kaappi/kaappi
[ -f zig-out/bin/kaappi ] || zig build
```

### 2. Determine which repos to test

If the user specified a repo name in args, test only that repo. Otherwise test all.

### 3. Run tests

For each repo, follow its category:

**Pure Scheme (no build, no deps):**
kaappi-cli, kaappi-json, kaappi-csv, kaappi-toml, kaappi-yaml, kaappi-log, kaappi-template, kaappi-test

```bash
KAAPPI=/Users/bmuthuka/kaappi/kaappi/zig-out/bin/kaappi
REPO=/Users/bmuthuka/kaappi/<repo-name>
for f in $REPO/tests/*.scm; do
  echo "=== $(basename $f) ==="
  $KAAPPI --lib-path $REPO/lib "$f"
done
```

**Native build, no deps:**
kaappi-net, kaappi-crypto, kaappi-sqlite

```bash
make -C $REPO
DYLD_LIBRARY_PATH=$REPO $KAAPPI --lib-path $REPO/lib $REPO/tests/*.scm
```

**With dependencies:**

| Repo | Deps to build | Lib paths | Native paths |
|------|---------------|-----------|--------------|
| kaappi-email | kaappi-net (make) | kaappi-net/lib | kaappi-net |
| kaappi-http | kaappi-net (make), self (make) | kaappi-net/lib, self/lib | kaappi-net, self |
| kaappi-redis | kaappi-net (make), self (make) | kaappi-net/lib, self/lib | kaappi-net, self |
| kaappi-pg | self (make) | self/lib | self |
| kaappi-web | kaappi-net (make), kaappi-http, kaappi-json | all four /lib paths | kaappi-net |

Build deps first, then construct --lib-path and DYLD_LIBRARY_PATH accordingly.

### 4. Run with --coverage if user asks for coverage

Add `--coverage` to the kaappi invocation. Add `--coverage-xml /tmp/<repo>-coverage.xml` if they want XML output.

### 5. Report results

Summarize pass/fail for each repo tested.
