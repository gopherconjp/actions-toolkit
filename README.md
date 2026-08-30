# actions-toolkit

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Verify (Repo)](https://github.com/gopherconjp/actions-toolkit/actions/workflows/verify-repo.yaml/badge.svg)](https://github.com/gopherconjp/actions-toolkit/actions/workflows/verify-repo.yaml)
[![CodeQL Advanced](https://github.com/gopherconjp/actions-toolkit/actions/workflows/codeql.yaml/badge.svg)](https://github.com/gopherconjp/actions-toolkit/actions/workflows/codeql.yaml)

A collection of reusable workflows and composite actions

## Reusable Workflows

### verify-actions

Lints GitHub Actions (workflows / composite actions) with actionlint, ghalint, and zizmor.

```yaml
name: Verify (Actions)

on:
  pull_request:

jobs:
  verify-actions:
    uses: gopherconjp/actions-toolkit/.github/workflows/verify-actions.yaml@main
    permissions:
      contents: read
      checks: write
```

Grant the `contents: read` and `checks: write` permissions on the calling job.

## Composite Actions

### setup-bun

Sets up JS runtimes (via mise) and installs dependencies with Bun.

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v7
  - name: Setup Bun environment
    uses: gopherconjp/actions-toolkit/setup-bun@main
```

### wait-for-workflow

Waits for another workflow run on the same commit to complete, failing if it does not succeed.  
Requires the `actions: read` permission.

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v7
  - name: Wait for tests
    uses: gopherconjp/actions-toolkit/wait-for-workflow@main
    with:
      workflow-id: test.yaml
      timeout-minutes: 15 # defaults to 10
```

#### Inputs

| Input             | Required | Default | Description                          |
| ----------------- | -------- | ------- | ------------------------------------ |
| `workflow-id`     | ✅       | —       | Workflow file name or ID to wait for |
| `timeout-minutes` | —        | `10`    | Maximum time to wait in minutes      |
