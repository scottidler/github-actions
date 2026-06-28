# github-actions

Reusable GitHub Actions workflows for `scottidler` Rust projects.

This is the Rust-only subset twin of `tatari-tv/github-actions`: the two Rust
reusable workflows are kept byte-identical across the two orgs so a repo in
either org calls the same pipeline from its own org. Everything tatari-specific
(ECR, CodeArtifact, docker-build, deploy dispatch) is intentionally omitted here.

## Workflows

### `rust-ci.yml`

Push/PR CI for Rust CLIs. One job: runs the repo's checks via `otto ci`
(whitespace, clippy, fmt, test) on a prepared toolchain. Caller owns the trigger.

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
permissions:
  contents: read
jobs:
  ci:
    uses: scottidler/github-actions/.github/workflows/rust-ci.yml@v1
    with:
      rust-version: "1.96.0"
```

### `rust-cli-release.yml`

Tag-driven CLI release. Cross-compiles the requested targets (linux amd64/arm64,
macOS x86_64/arm64), tarballs + sha256 each, and publishes a GitHub Release.

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags:
      - 'v*'
  workflow_dispatch: {}
permissions:
  contents: write
jobs:
  release:
    uses: scottidler/github-actions/.github/workflows/rust-cli-release.yml@v1
    with:
      rust-version: "1.96.0"
```

## Inputs

Both workflows accept `rust-version` (toolchain) and `private-deps` (+ the
`private-deps-token` secret) for private git dependencies. `rust-cli-release.yml`
also accepts `bin-names`, `archive-prefix`, `targets`, and `extra-files` - all
defaulting to sensible values derived from the repo name. See each workflow's
`workflow_call.inputs` block for the full contract.

## Versioning

Pin callers to a tag (`@v1`). These files are generated into new projects by
[`scaffold`](https://github.com/scottidler/scaffold).
