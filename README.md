# github-workflows

> Reusable GitHub Actions workflows for **Rust** and **Dioxus** projects.
> Instead of copy-pasting 200+ lines of YAML into every repo, each repo just calls one of these workflows and passes a
> couple of inputs.

---

## 📚 Overview

This repository provides four reusable building blocks plus one legacy pair:

| Building block                  | What it does                                                                                           |
|---------------------------------|--------------------------------------------------------------------------------------------------------|
| **`release-plz.yml`**           | Automated semantic versioning + crates.io publishing via [release-plz](https://release-plz.ieni.dev/). |
| **`build-assets.yml`**          | Builds release binaries and uploads them as artifacts. **Does not create a release.**                  |
| **`release-assets.yml`**        | Downloads artifacts from a prior build and creates a GitHub Release.                                   |
| **`build-dioxus-assets.yml`**   | Builds Dioxus desktop bundles (deb/AppImage/rpm/dmg/msi) and uploads them as artifacts.                |
| **`release-dioxus-assets.yml`** | Downloads Dioxus artifacts and creates a GitHub Release.                                               |

> **Design note:** Build and Release are intentionally **separated**. The build job uploads artifacts; the release job
> consumes them. This lets you build on every PR/push but only release on tags, and keeps `contents: read` for builds vs
`contents: write` for releases.

---

## 🗺️ Workflow Map

| File                        | Purpose                                  | Status            |
|-----------------------------|------------------------------------------|-------------------|
| `release-plz.yml`           | Semantic release / publish               | ✅ Active          |
| `build-assets.yml`          | Build binaries (normal Rust)             | ✅ Active          |
| `release-assets.yml`        | Create GitHub Release (normal Rust)      | ✅ Active          |
| `build-dioxus-assets.yml`   | Build Dioxus desktop bundles             | ✅ Active          |
| `release-dioxus-assets.yml` | Create GitHub Release (Dioxus)           | ✅ Active          |
| `build-binaries.yml`        | Old combined build+release (normal Rust) | ⚠️ **Deprecated** |
| `build-dioxus-binaries.yml` | Old combined build+release (Dioxus)      | ⚠️ **Deprecated** |

---

## 🚀 Quick Start

### 1️⃣ Release-plz (semantic versioning + publishing)

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches:
      - main

jobs:
  release:
    uses: seyallius/github-workflows/.github/workflows/release-plz.yml@v1
    with:
      branch: main                 # Use 'master' if that is your default branch
    secrets:
      RELEASE_PLZ_TOKEN: ${{ secrets.RELEASE_PLZ_TOKEN }}
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### 2️⃣ Build & Release — Normal Rust binaries

```yaml
# .github/workflows/build-release.yml
name: Build & Release
on:
  push:
    tags:
      - "v*"

# Required so the release job can create a GitHub Release.
permissions:
  contents: write

jobs:
  # Step 1: build binaries and upload artifacts (no release created here).
  build:
    uses: seyallius/github-workflows/.github/workflows/build-assets.yml@v1
    with:
      binary_name: treeclip        # <-- CHANGE THIS to your crate/binary name

  # Step 2: download the artifacts from `build` and publish a GitHub Release.
  release:
    needs: build
    uses: seyallius/github-workflows/.github/workflows/release-assets.yml@v1
    with:
      artifact_pattern: treeclip* # Must match what the build step uploaded
```

### 3️⃣ Build & Release — Dioxus desktop app

```yaml
# .github/workflows/build-release-dioxus.yml
name: Build & Release (Dioxus)
on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-dioxus-assets.yml@v1
    with:
      app_name: my-dioxus-app      # <-- CHANGE THIS to your Dioxus app name
      build_linux: true
      build_macos: true
      build_windows: true

  release:
    needs: build
    uses: seyallius/github-workflows/.github/workflows/release-dioxus-assets.yml@v1
    with:
      app_name: my-dioxus-app
```

---

## 🔧 Workflow Reference

### 🔄 release-plz.yml

Automates version bumps, changelog, git tagging, and crates.io publishing.

| Input    | Type   | Default | Required | Description                   |
|----------|--------|---------|----------|-------------------------------|
| `branch` | string | `main`  | no       | Branch to watch for releases. |

| Secret                 | Required | Description                                                  |
|------------------------|----------|--------------------------------------------------------------|
| `RELEASE_PLZ_TOKEN`    | ✅        | Personal Access Token with `repo` + `write:packages` scopes. |
| `CARGO_REGISTRY_TOKEN` | ✅        | Your crates.io API token.                                    |

> ⚠️ **Important:** Both jobs in this workflow are gated by
> `if: ${{ github.repository_owner == 'seyallius' }}`. If you reuse this
> workflow from an account/organization **other than `seyallius`**, the jobs
> will be **skipped**. Adjust that guard in your fork if you need it to run
> elsewhere.

---

### 🏗️ build-assets.yml (normal Rust)

Builds binaries for each target and uploads them as artifacts. **Does not create a release.**

| Input           | Type   | Default                                                               | Required | Description                                                  |
|-----------------|--------|-----------------------------------------------------------------------|----------|--------------------------------------------------------------|
| `binary_name`   | string | —                                                                     | ✅        | Name of the binary/crate to build.                           |
| `build_command` | string | `cargo build --release`                                               | no       | Custom build command. `${{ matrix.target }}` is substituted. |
| `targets`       | string | `x86_64-unknown-linux-gnu,x86_64-pc-windows-msvc,x86_64-apple-darwin` | no       | Comma-separated Rust targets.                                |

| Output           | Description                      |
|------------------|----------------------------------|
| `artifact_names` | Names of the artifacts produced. |

- **Permissions needed:** `contents: read`.
- **Artifact naming:** `{binary_name}-{version}-{target}.tar.gz` (or `.zip` on Windows).

---

### 📦 release-assets.yml (normal Rust)

Downloads artifacts matching a pattern and creates a GitHub Release.

| Input                    | Type    | Default           | Required | Description                                  |
|--------------------------|---------|-------------------|----------|----------------------------------------------|
| `artifact_pattern`       | string  | —                 | ✅        | Pattern to match artifacts (e.g. `myapp-*`). |
| `tag_name`               | string  | `github.ref_name` | no       | Git tag to release from.                     |
| `generate_release_notes` | boolean | `true`            | no       | Auto-generate release notes.                 |
| `draft`                  | boolean | `false`           | no       | Create as a draft release.                   |
| `prerelease`             | boolean | `false`           | no       | Mark as a prerelease.                        |

- **Permissions needed:** `contents: write`.

---

### 🖥️ build-dioxus-assets.yml (Dioxus)

Builds Dioxus desktop bundles and uploads them as artifacts. **Does not create a release.**

| Input           | Type    | Default | Required | Description                        |
|-----------------|---------|---------|----------|------------------------------------|
| `app_name`      | string  | —       | ✅        | Name of the Dioxus app.            |
| `build_linux`   | boolean | `true`  | no       | Build `.deb`, `.AppImage`, `.rpm`. |
| `build_macos`   | boolean | `true`  | no       | Build `.dmg`.                      |
| `build_windows` | boolean | `true`  | no       | Build `.msi`.                      |

- **Permissions needed:** `contents: read`.
- **Artifacts produced:** `linux-deb`, `linux-appimage`, `linux-rpm`, `macos-dmg`, `windows-msi`.

---

### 🎁 release-dioxus-assets.yml (Dioxus)

Downloads all Dioxus artifacts from the build step and creates a GitHub Release.

| Input                    | Type    | Default           | Required | Description                  |
|--------------------------|---------|-------------------|----------|------------------------------|
| `app_name`               | string  | —                 | ✅        | Name of the Dioxus app.      |
| `tag_name`               | string  | `github.ref_name` | no       | Git tag to release from.     |
| `generate_release_notes` | boolean | `true`            | no       | Auto-generate release notes. |
| `draft`                  | boolean | `false`           | no       | Create as a draft release.   |
| `prerelease`             | boolean | `false`           | no       | Mark as a prerelease.        |

- **Permissions needed:** `contents: write`.
- **Attached assets:** `*.deb`, `*.AppImage`, `*.rpm`, `*.dmg`, `*.msi`.

---

## ⚠️ Deprecated Workflows & Migration

`build-binaries.yml` and `build-dioxus-binaries.yml` are **deprecated**. They emit a deprecation warning and will fail
after their enforcement date. Migrate to the split workflows:

| Deprecated workflow         | Replace with                                                |
|-----------------------------|-------------------------------------------------------------|
| `build-binaries.yml`        | `build-assets.yml` **+** `release-assets.yml`               |
| `build-dioxus-binaries.yml` | `build-dioxus-assets.yml` **+** `release-dioxus-assets.yml` |

**Before (deprecated, single combined workflow):**

```yaml
jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-binaries.yml@v1
    with:
      binary_name: treeclip
```

**After (split build + release):**

```yaml
permissions:
  contents: write

jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-assets.yml@v1
    with:
      binary_name: treeclip

  release:
    needs: build
    uses: seyallius/github-workflows/.github/workflows/release-assets.yml@v1
    with:
      artifact_pattern: treeclip-*
```

> The key difference: the old workflow created the release internally on tag push. The new model gives **you** control
> over *when* the release runs (via the caller's `on:` trigger) and splits permissions cleanly.

If you temporarily must keep using `build-binaries.yml`, it accepts `ignore_deprecation: true` to suppress the warning
and `enforce_migration_date` (default `2027-01-01`) to control when it hard-fails. Treat these as a short-term bridge
only.

---

## 🔑 Per-Repository Secrets & Variables

### Secrets (Settings → Secrets and variables → Actions → Secrets)

| Secret                 | Used by           | Description                         |
|------------------------|-------------------|-------------------------------------|
| `RELEASE_PLZ_TOKEN`    | `release-plz.yml` | PAT with `repo` + `write:packages`. |
| `CARGO_REGISTRY_TOKEN` | `release-plz.yml` | crates.io API token.                |

> The build/release **asset** workflows do **not** require custom secrets — they use the default `GITHUB_TOKEN`, which
> is passed automatically to reusable workflows. Just make sure the caller grants the right `permissions`.

### Variables (optional)

Prefer setting non-sensitive config via the GitHub UI? Add a repository variable and reference it:

```yaml
with:
  binary_name: ${{ vars.BINARY_NAME }}
```

---

## 🎯 Summary Table

| Workflow                    | Things that change per repo              | How to provide them        |
|-----------------------------|------------------------------------------|----------------------------|
| `release-plz.yml`           | None (crate name read from `Cargo.toml`) | Secrets only               |
| `build-assets.yml`          | `binary_name`                            | Input or repo variable     |
| `release-assets.yml`        | `artifact_pattern`                       | Input (match build output) |
| `build-dioxus-assets.yml`   | `app_name`                               | Input or repo variable     |
| `release-dioxus-assets.yml` | `app_name`                               | Input                      |

---

## 📁 Repository Structure

```
.github/
└── workflows/
    ├── build-assets.yml            # ✅ Build binaries (normal Rust)
    ├── release-assets.yml          # ✅ Release binaries (normal Rust)
    ├── build-dioxus-assets.yml     # ✅ Build Dioxus bundles
    ├── release-dioxus-assets.yml   # ✅ Release Dioxus bundles
    ├── release-plz.yml             # ✅ Semantic release / publish
    ├── build-binaries.yml          # ⚠️ Deprecated (normal Rust)
    └── build-dioxus-binaries.yml   # ⚠️ Deprecated (Dioxus)
```
