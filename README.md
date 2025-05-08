# workflow-rust-release

Reusable GitHub Actions workflows for Rust CLI release automation.

## Overview

This directory provides a set of workflows to automate the versioning, tagging, release, and optional registry publishing (crates.io, Homebrew, Scoop) for Rust CLI tools.

### Workflows Included

- **on-push-main-version-and-tag.yaml**
  _Trigger:_ Push to `main`
  _Purpose:_ Determines the next version number based on commit logs and tags, and creates a new version tag for release. Uses the reusable workflow from `workflow-vnext-tag`.
  _Typical Use:_ Automates semantic versioning and tagging for your project.

- **on-v-tag-release.yaml**
  _Trigger:_ Push of a tag matching `v*.*.*`
  _Purpose:_ Creates a GitHub Release for the new version tag using the ncipollo/release-action.
  _Typical Use:_ Ensures every version tag gets a corresponding GitHub Release.

- **workflow.yaml**
  _Trigger:_ `workflow_call` (intended to be called from other repositories or workflows)
  _Purpose:_ Builds Rust binaries for a matrix of platforms, uploads them to the GitHub Release, and optionally publishes to crates.io, Homebrew, and Scoop.
  _Typical Use:_ The main reusable workflow for cross-platform Rust CLI release automation.

---

## Typical Release Flow

1. **Push to main**
   - `on-push-main-version-and-tag.yaml` runs, determines the next version, and creates a tag (e.g., `v1.2.3`).

2. **Tag is pushed**
   - `on-v-tag-release.yaml` runs, creating a GitHub Release for the new tag.

3. **Reusable workflow is called**
   - `workflow.yaml` is called (manually or via another workflow) to build binaries, upload them to the release, and optionally publish to registries.

---

## Usage: Reusable Release Workflow

In your Rust CLI repository, add a workflow like:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    uses: harmony-labs/workflow-rust-release/.github/workflows/workflow.yaml@main
    with:
      binary_name: gh-config
      crate_name: gh-config-cli
      homebrew_tap: harmony-labs/homebrew-tap
      scoop_bucket: harmony-labs/scoop-bucket
    secrets:
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
      HOMEBREW_GITHUB_TOKEN: ${{ secrets.HOMEBREW_GITHUB_TOKEN }}
      SCOOP_GITHUB_TOKEN: ${{ secrets.SCOOP_GITHUB_TOKEN }}
```

- Omit any input or secret to skip that registry/platform.

## Inputs

- `binary_name` (required): Name of the built binary/executable.
- `crate_name` (optional): Name of the crate for crates.io publishing.
- `homebrew_tap` (optional): Homebrew tap repo (org/repo) for formula PR.
- `scoop_bucket` (optional): Scoop bucket repo (org/repo) for manifest PR.
- `platforms` (optional): JSON array of build targets (see workflow for default).

## Secrets

- `CARGO_REGISTRY_TOKEN` (optional): crates.io API token.
- `HOMEBREW_GITHUB_TOKEN` (optional): GitHub token with write access to the Homebrew tap repo.
- `SCOOP_GITHUB_TOKEN` (optional): GitHub token with write access to the Scoop bucket repo.

## How Homebrew/Scoop Updates Work

- The workflow uses the GitHub CLI (`gh`) to fork, clone, update, commit, and open a PR to the tap/bucket repo.
- You must implement the logic to update the formula/manifest file in the repo (see TODOs in the workflow).

## Extending

- Add logic to update the Homebrew formula and Scoop manifest as needed for your project.
- You can customize the build matrix by overriding the `platforms` input.

## License

MIT
