# actions-toolkit

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Verify (Repo)](https://github.com/logica-oss/actions-toolkit/actions/workflows/verify-repo.yaml/badge.svg)](https://github.com/logica-oss/actions-toolkit/actions/workflows/verify-repo.yaml)
[![CodeQL Advanced](https://github.com/logica-oss/actions-toolkit/actions/workflows/codeql.yaml/badge.svg)](https://github.com/logica-oss/actions-toolkit/actions/workflows/codeql.yaml)

A collection of composite actions

## Composite Actions

### verify-actions

Lints GitHub Actions (workflows / composite actions) with actionlint, ghalint, and zizmor.  
Requires the `contents: read` and `checks: write` permissions.

```yaml
jobs:
  verify-actions:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: read
      checks: write
    steps:
      - name: Checkout
        uses: actions/checkout@v7
        with:
          persist-credentials: false
      - name: Verify actions
        uses: logica-oss/actions-toolkit/verify-actions@main
```

### setup-bun

Sets up JS runtimes (via mise) and installs dependencies with Bun.  
No special permissions are required.

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Checkout
        uses: actions/checkout@v7
      - name: Setup Bun environment
        uses: logica-oss/actions-toolkit/setup-bun@main
```

### wait-for-workflow

Waits for another workflow run on the same commit to complete, failing if it does not succeed.  
Requires the `contents: read` and `actions: read` permissions.

```yaml
jobs:
  wait:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: read
      actions: read
    steps:
      - name: Checkout
        uses: actions/checkout@v7
      - name: Wait for tests
        uses: logica-oss/actions-toolkit/wait-for-workflow@main
        with:
          workflow-id: test.yaml
          timeout-minutes: 15 # defaults to 10
```

#### Inputs

| Input             | Required | Default | Description                          |
| ----------------- | -------- | ------- | ------------------------------------ |
| `workflow-id`     | ✅       | —       | Workflow file name or ID to wait for |
| `timeout-minutes` | —        | `10`    | Maximum time to wait in minutes      |

### check-release-label

Fails unless exactly one of the patch, minor, or major release labels is attached.  
Requires the `pull-requests: read` permission.

```yaml
name: Verify (Release Label)

on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]

jobs:
  check-release-label:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions:
      pull-requests: read
    steps:
      - name: Check release label
        uses: logica-oss/actions-toolkit/check-release-label@main
        with:
          major-label: major # defaults to major
          minor-label: minor # defaults to minor
          patch-label: patch # defaults to patch
          ignore-authors: | # defaults to renovate[bot]
            renovate[bot]
```

#### Inputs

| Input            | Required | Default         | Description                                  |
| ---------------- | -------- | --------------- | -------------------------------------------- |
| `major-label`    | —        | `major`         | Label triggering a major release             |
| `minor-label`    | —        | `minor`         | Label triggering a minor release             |
| `patch-label`    | —        | `patch`         | Label triggering a patch release             |
| `ignore-authors` | —        | `renovate[bot]` | Authors skipped without labels, one per line |

### release

Creates a SemVer tag and GitHub Release from merged PR labels.  
Requires the `contents: write` and `pull-requests: read` permissions.

```yaml
name: Release

on:
  schedule:
    - cron: "0 0 * * 1"
  workflow_dispatch:

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: write
      pull-requests: read
    steps:
      - name: Checkout
        uses: actions/checkout@v7
      - name: Create release
        id: release
        uses: logica-oss/actions-toolkit/release@main
        with:
          initial-version: v1.0.0 # defaults to v1.0.0
          major-label: major # defaults to major
          minor-label: minor # defaults to minor
```

The `minor` label triggers a minor release.  
The `major` label triggers a major release.  
PRs without either label default to a patch release.

#### Inputs

| Input             | Required | Default  | Description                                 |
| ----------------- | -------- | -------- | ------------------------------------------- |
| `initial-version` | —        | `v1.0.0` | Tag created when no previous release exists |
| `major-label`     | —        | `major`  | Label triggering a major release            |
| `minor-label`     | —        | `minor`  | Label triggering a minor release            |

#### Outputs

| Output | Description                                     |
| ------ | ----------------------------------------------- |
| `tag`  | Created tag, empty when no release was created. |

### sync-agent-config

Syncs agent configs from canonical sources.  
Requires the `contents: read` permission.  
`.claude/rules` and `.claude/skills` are fully regenerated on each run; do not place hand-written files there.

Canonical sources and generated mirrors:

| Source                                   | Mirror                                     |
| ---------------------------------------- | ------------------------------------------ |
| `AGENTS.md`                              | `.github/copilot-instructions.md`          |
| `.github/instructions/*.instructions.md` | `.claude/rules/*.md` (`applyTo` → `paths`) |
| `.agents/skills/*`                       | `.claude/skills/*` (copy)                  |

```yaml
name: autofix.ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  autofix:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v7
        with:
          persist-credentials: false
      - name: Sync Agent Config
        uses: logica-oss/actions-toolkit/sync-agent-config@main
      - name: Autofix
        uses: autofix-ci/action@v1
```

### autofix-docs

Fixes Markdown with markdownlint, formats docs with Oxfmt, commits via autofix-ci, then re-lints.  
Requires the `contents: read` permission and the autofix.ci GitHub App.  
The calling workflow's `name` must be `autofix.ci` (required by autofix-ci).

```yaml
name: autofix.ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  autofix:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v7
        with:
          persist-credentials: false
      - name: Autofix Docs
        uses: logica-oss/actions-toolkit/autofix-docs@main
```

#### Inputs

| Input            | Required | Default                                                                              | Description                                |
| ---------------- | -------- | ------------------------------------------------------------------------------------ | ------------------------------------------ |
| `markdown-globs` | —        | `**/*.{md,markdown}`                                                                 | Markdown files to lint, newline-delimited  |
| `oxfmt-paths`    | —        | `**/*.md **/*.markdown **/*.yaml **/*.yml **/*.json **/*.jsonc **/*.json5 **/*.toml` | Paths for Oxfmt to format, space-delimited |
| `enable-oxfmt`   | —        | `true`                                                                               | Whether to run Oxfmt formatting            |
