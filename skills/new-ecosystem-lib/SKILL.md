---
name: new-ecosystem-lib
description: Scaffold a new kaappi-* ecosystem library with standard layout, CI, and tests. Usage /new-ecosystem-lib <name> [--ffi] [--depends dep1 dep2]
---

# New Ecosystem Library

Scaffold a new `kaappi-*` ecosystem library with the standard file layout,
CI workflow, and test skeleton.

## Arguments

1. `<name>` (required) — library name without the `kaappi-` prefix (e.g. `auth`)
2. `--ffi` (optional) — include C FFI scaffolding (`csrc/`, `Makefile`)
3. `--depends <deps>` (optional) — space-separated dependency names (e.g. `kaappi-net kaappi-json`)

## Steps

### 1. Create the repo directory

```bash
mkdir -p /Users/bmuthuka/kaappi/kaappi-<name>
cd /Users/bmuthuka/kaappi/kaappi-<name>
```

### 2. Create `kaappi.pkg`

For pure Scheme:
```
name: kaappi-<name>
kaappi-version: >=0.1.0
```

With dependencies:
```
name: kaappi-<name>
depends: <dep1> <dep2>
kaappi-version: >=0.1.0
```

With FFI (add `build: make`):
```
name: kaappi-<name>
build: make
depends: <dep1> <dep2>
kaappi-version: >=0.1.0
```

### 3. Create `lib/kaappi/<name>.sld`

```bash
mkdir -p lib/kaappi
```

Write the library definition:
```scheme
(define-library (kaappi <name>)
  (import (scheme base))
  (export )
  (begin
    ))
```

### 4. Create `tests/test-<name>.scm`

```bash
mkdir -p tests
```

Write the test skeleton:
```scheme
(import (scheme base)
        (scheme write)
        (kaappi <name>))

(import (chibi test))

(test-begin "kaappi-<name>")

;; Add tests here

(test-end "kaappi-<name>")
```

### 5. If `--ffi`: create C FFI scaffolding

Create `csrc/kaappi_<name>.c`:
```bash
mkdir -p csrc
```

```c
#include <stdlib.h>

/* C helper functions for kaappi-<name> FFI */
```

Create `Makefile`:
```makefile
UNAME := $(shell uname)
ifeq ($(UNAME), Darwin)
  DYLIB_EXT := dylib
  CFLAGS_SHARED := -dynamiclib
else
  DYLIB_EXT := so
  CFLAGS_SHARED := -shared -fPIC
endif

CC ?= cc
CFLAGS := -O2 -Wall -Wextra

.PHONY: all clean

all: libkaappi_<name>.$(DYLIB_EXT)

libkaappi_<name>.$(DYLIB_EXT): csrc/kaappi_<name>.c
	$(CC) $(CFLAGS) $(CFLAGS_SHARED) -o $@ $<

clean:
	rm -f libkaappi_<name>.dylib libkaappi_<name>.so
```

If the library needs system dependencies (e.g. OpenSSL, libpq), add
`pkg-config` flags to the Makefile following the `kaappi-net` pattern.

### 6. Create `.github/workflows/ci.yml`

```bash
mkdir -p .github/workflows
```

For pure Scheme:
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install Zig
        uses: mlugg/setup-zig@v2
        with:
          version: 0.16.0

      - name: Build Kaappi
        run: |
          git clone --depth 1 https://github.com/kaappi/kaappi.git /tmp/kaappi
          cd /tmp/kaappi && zig build

      - name: Run tests
        run: /tmp/kaappi/zig-out/bin/kaappi --coverage-xml coverage.xml --lib-path lib tests/test-<name>.scm

      - name: Upload to Codecov
        if: github.event_name == 'push'
        uses: codecov/codecov-action@v5
        with:
          use_oidc: true
          files: coverage.xml
          fail_ci_if_error: false
```

For FFI libraries, add a "Build native library" step before "Run tests":
```yaml
      - name: Build native library
        run: make
```

For libraries with dependencies, clone and build each dependency before
running tests. Follow the `kaappi-http` CI pattern.

### 7. Create `.gitignore`, `README.md`, `LICENSE`

`.gitignore`:
```
*.dylib
*.so
*.o
```

`README.md`:
```markdown
# kaappi-<name>

<One-line description>.

## Installation

```
thottam install kaappi-<name>
```

## Usage

```scheme
(import (kaappi <name>))
```

## License

MIT
```

`LICENSE` — create an MIT license file with the current year and "Kaappi
Contributors" as the copyright holder.

### 8. Initialize git repo

```bash
git init
git add .
git commit -m "Initial scaffold"
```

### 9. Nightly workflow entry

Print the matrix entry to add to `.github/.github/workflows/nightly.yml`.
Do NOT auto-edit the nightly file — it lives in a different repo.

For pure Scheme (add to `pure-libs` group):
```yaml
        - repo: kaappi-<name>
          test-files: tests/test-<name>.scm
```

For FFI without services (add to `native-libs` group):
```yaml
        - repo: kaappi-<name>
          has-native-build: true
          test-files: tests/test-<name>.scm
```

For libraries with dependencies (add to `dep-libs` group):
```yaml
        - repo: kaappi-<name>
          dependencies: <dep1> <dep2>
          test-files: tests/test-<name>.scm
```

Tell the user which group and the exact YAML block to paste.

### 10. Verify

```bash
cd /Users/bmuthuka/kaappi/kaappi-<name>
/Users/bmuthuka/kaappi/kaappi/zig-out/bin/kaappi --lib-path lib tests/test-<name>.scm
```

The empty test suite should pass with 0 tests.
