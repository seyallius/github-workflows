# github-workflows

My reusable github action workflows.

# How to use

## 🔄 **1. Release-plz (Reusable)**

Setup semantic release with release-plz.

**Changes made:**

- Changed `on: push` to `workflow_call`
- Added `inputs` and `secrets` configuration
- Removed hardcoded `main` branch (now configurable)
- All secrets must be passed explicitly

### **Calling it from any repository** (`.github/workflows/release.yml`):

```yaml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    uses: seyallius/github-workflows/.github/workflows/release-plz.yml@v1
    with:
      branch: main  # Or 'master' if you use that
    secrets:
      RELEASE_PLZ_TOKEN: ${{ secrets.RELEASE_PLZ_TOKEN }}
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

**What changes per repository?**

- You need to set `RELEASE_PLZ_TOKEN` and `CARGO_REGISTRY_TOKEN` as **repository secrets** in each repo (or organization
  secrets)
- The branch name might vary (if you use `master` instead of `main`)

---

## 🏗️ **2. Build Binaries (Normal Rust)**

Build cargo binaries.

### **Calling it from any repository** (`.github/workflows/build.yml`):

```yaml
name: Build & Release

on:
  push:
    tags:
      - "v*"

jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-binaries.yml@v1
    with:
      binary_name: treeclip  # IMPORTANT: Change this per repository!
      # Optional: Customize targets if needed
      targets: 'x86_64-unknown-linux-gnu,x86_64-pc-windows-msvc,x86_64-apple-darwin'
```

**What changes per repository?**

- **`binary_name`**: The name of your crate/binary (e.g., `treeclip`, `my-tool`, etc.)
- **`targets`**: If you need different targets (e.g., only Linux and Windows)

---

## 🖥️ **3. Build Binaries (Dioxus Version)**

Build dx binaries.

### **Calling it from a Dioxus repository** (`.github/workflows/build-dioxus.yml`):

```yaml
name: Build Dioxus App

on:
  push:
    tags:
      - "v*"

jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-dioxus-binaries.yml@v1
    with:
      app_name: my-dioxus-app  # CHANGE THIS per repository!
      build_linux: true
      build_macos: true
      build_windows: true
```

**What changes per repository?**

- **`app_name`**: The name of your Dioxus app (used in naming)
- **`build_linux`/`build_macos`/`build_windows`**: Toggle platforms if needed

---

## 🔑 **Per-Repository Secrets & Variables**

### **For Each Repository:**

1. **Secrets** (for sensitive data):
    - `RELEASE_PLZ_TOKEN`: A Personal Access Token with `repo` and `write:packages` scopes
    - `CARGO_REGISTRY_TOKEN`: Your crates.io token (if publishing)

2. **Repository Variables** (for non-sensitive config):
    - You can also use repository variables for the binary name, but I recommend passing it as an input

### **Using Repository Variables instead of inputs**:

If you prefer to set these in the GitHub UI:

1. Go to Settings → Secrets and variables → Actions → Variables
2. Add a variable like `BINARY_NAME`
3. In your caller workflow:

```yaml
with:
  binary_name: ${{ vars.BINARY_NAME }}
```

---

## 🎯 **Summary Table**

| Workflow            | Changing Variables                     | Method                       |
|---------------------|----------------------------------------|------------------------------|
| **Release-plz**     | None (uses crate name from Cargo.toml) | Secrets only                 |
| **Build Binaries**  | `binary_name` (crate name)             | Input or Repository Variable |
| **Dioxus Binaries** | `app_name` (app name)                  | Input or Repository Variable |

---

## 📝 **One-Liner for Each Repository**

Now, instead of copy-pasting 200+ lines, each repository just needs:

**For normal Rust projects:**

```yaml
# .github/workflows/release.yml
on: push
jobs:
  release:
    uses: seyallius/github-workflows/.github/workflows/release-plz.yml@v1
    secrets: inherit
```

**For binaries:**

```yaml
# .github/workflows/build.yml
on: push
jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-binaries.yml@v1
    with:
      binary_name: your-crate-name  # The only thing that changes!
```

**For Dioxus:**

```yaml
# .github/workflows/build-dioxus.yml
on: push
jobs:
  build:
    uses: seyallius/github-workflows/.github/workflows/build-dioxus-binaries.yml@v1
    with:
      app_name: your-app-name  # The only thing that changes!
```
