# sindarin-pipelines

Reusable GitHub Actions workflows for the Sindarin SDK ecosystem.

## Workflows

### `sindarin-pkg-test.yml` — Sindarin Package CI

For repos written in Sindarin (`.sn` source files) that use `sn --install` and `make test`.

**What it does:**
1. Installs platform build tools via the compiler's prereqs scripts
2. Installs the Sindarin compiler from the latest release
3. Runs `sn --install` to fetch package dependencies
4. Runs `make test`
5. On failure: re-compiles failing test files with verbose output

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

**Used by:** `sindarin-pkg-sdk`, `sindarin-pkg-http`, `sindarin-pkg-test`, `sindarin-pkg-json`

---

### `sindarin-lib-test.yml` — Native C Library CI

For native C libraries that use CMake + vcpkg and `make test` (cmake → ctest).

**What it does:**
1. Installs platform build tools via the compiler's prereqs scripts
2. Bootstraps vcpkg and sets `VCPKG_ROOT`
3. Caches `vcpkg_installed` keyed on `vcpkg.json`
4. Runs `make test` (cmake configure + build + ctest)
5. On failure: prints cmake/vcpkg versions and reruns ctest verbosely

**Requirements:** repo must have a `vcpkg.json` and a `Makefile` with a `test` target that invokes cmake with the vcpkg toolchain.

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
    uses: SindarinSDK/sindarin-pipelines/.github/workflows/sindarin-lib-test.yml@main
```

**Used by:** `sindarin-template`

---

## Platform Support

Both workflows run on **Ubuntu**, **macOS**, and **Windows** in parallel.

| Tool | Linux | macOS | Windows |
|------|-------|-------|---------|
| C compiler | gcc (`build-essential`) | Xcode clang | LLVM-MinGW |
| make | included | included | Chocolatey |
| cmake | pre-installed | pre-installed | pre-installed |
| vcpkg | — | — | bootstrapped (lib workflow only) |
