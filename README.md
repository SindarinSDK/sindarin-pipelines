# sindarin-pipelines

Reusable GitHub Actions workflows and composite actions for the Sindarin SDK ecosystem.

## Composite Actions

Shared building blocks used by workflows and package-specific CI.

### `setup-prereqs`

Installs platform build tools (gcc/clang, make, cmake) via the Sindarin compiler prereqs scripts.

```yaml
- uses: SindarinSDK/sindarin-pipelines/.github/actions/setup-prereqs@main
```

### `install-sindarin`

Downloads and installs the Sindarin compiler from the latest GitHub release.

```yaml
- uses: SindarinSDK/sindarin-pipelines/.github/actions/install-sindarin@main
  with:
    github-token: ${{ github.token }}
```

### `install-deps`

Runs `sn --install` to fetch declared package dependencies.

```yaml
- uses: SindarinSDK/sindarin-pipelines/.github/actions/install-deps@main
  with:
    github-token: ${{ github.token }}
```

### `diagnose-pkg`

Re-compiles all `tests/test_*.sn` files with verbose output to surface compile errors on failure.

```yaml
- name: Diagnose compile errors
  if: failure()
  uses: SindarinSDK/sindarin-pipelines/.github/actions/diagnose-pkg@main
```

---

## Reusable Workflows

### `sindarin-pkg-test.yml` — Sindarin Package CI

For repos written in Sindarin (`.sn` source files) that use `sn --install` and `make test`.

**Usage** — add `.github/workflows/ci.yml` to your repo:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: SindarinSDK/sindarin-pipelines/.github/workflows/sindarin-pkg-test.yml@main
```

**Platforms:** Ubuntu, macOS, Windows

**Used by:** `sindarin-pkg-sdk`, `sindarin-pkg-http`, `sindarin-pkg-test`, `sindarin-pkg-json`, `sindarin-pkg-sqlite`, `sindarin-pkg-curl`

---

### `sindarin-lib-test.yml` — Native C Library CI

For native C libraries that use CMake + vcpkg and `make test` (cmake → ctest).

**Usage:**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: SindarinSDK/sindarin-pipelines/.github/workflows/sindarin-lib-test.yml@main
```

**Platforms:** Ubuntu, macOS, Windows

**Used by:** `sindarin-template`

---

## Packages with External Services

Packages that require external services (databases, etc.) define their own job in `.github/workflows/ci.yml` and use the composite actions directly. This gives full control over service configuration while still sharing the common install/test steps.

**Example** — package requiring PostgreSQL:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: SindarinSDK/sindarin-pipelines/.github/actions/setup-prereqs@main
      - uses: SindarinSDK/sindarin-pipelines/.github/actions/install-sindarin@main
        with:
          github-token: ${{ github.token }}
      - uses: SindarinSDK/sindarin-pipelines/.github/actions/install-deps@main
        with:
          github-token: ${{ github.token }}
      - name: Run tests
        env:
          PGHOST: localhost
          PGDATABASE: testdb
          PGUSER: postgres
          PGPASSWORD: postgres
        run: make test
      - if: failure()
        uses: SindarinSDK/sindarin-pipelines/.github/actions/diagnose-pkg@main
```

**Used by:** `sindarin-pkg-postgres`

---

## Platform Support

| Tool        | Linux                | macOS         | Windows         |
|-------------|----------------------|---------------|-----------------|
| C compiler  | gcc (`build-essential`) | Xcode clang | LLVM-MinGW      |
| make        | included             | included      | Chocolatey      |
| cmake       | pre-installed        | pre-installed | pre-installed   |
| vcpkg       | —                    | —             | bootstrapped (lib workflow only) |
