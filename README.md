# workflows-rust

Reusable GitHub Actions workflows for Rust quality checks, binary release automation, and Cargo crate publishing.

## Overview

This repository provides reusable workflows to automate quality checks and release processes for Rust projects.

### Workflows Included

- **quality.yaml**
  _Trigger:_ `workflow_call`
  _Purpose:_ Builds and tests a Rust project with customizable build and test arguments.
  _Typical Use:_ Ensures code quality before release by running builds and tests.

- **release.yaml**
  _Trigger:_ `workflow_call`
  _Purpose:_ Builds Rust binaries for a matrix of platforms and publishes them as GitHub Release artifacts.
  _Typical Use:_ Automates cross-platform binary builds and release artifact publishing for CLI tools.

- **publish.yaml**
  _Trigger:_ `workflow_call`
  _Purpose:_ Publishes a Rust crate to crates.io using `cargo publish`.
  _Typical Use:_ Automates publishing library crates when version tags are created.

## Usage: Quality Workflow

The `quality.yaml` workflow builds and tests your Rust project. Example usage in your repository:

```yaml
name: Quality Check

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  quality:
    uses: unbounded-tech/workflows-rust/.github/workflows/quality.yaml@main
    with:
      cargo_build_args: '--no-default-features --verbose'
      cargo_test_args: '--no-default-features --verbose'
      cargo_incremental: false
      runs-on: 'ubuntu-latest'
      verbose_logging: true
```

### Inputs for `quality.yaml`

- `cargo_build_args` (optional, default: `--no-default-features --verbose`): Additional arguments for `cargo build`.
- `cargo_test_args` (optional, default: `--no-default-features --verbose`): Additional arguments for `cargo test`.
- `cargo_incremental` (optional, default: `false`): Enable incremental compilation.
- `runs-on` (optional, default: `ubuntu-latest`): Runner label used for the job.
- `verbose_logging` (optional, default: `false`): Enable verbose logging.

## Usage: Release Workflow

The `release.yaml` workflow builds and publishes binaries. Example usage:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    uses: unbounded-tech/workflows-rust/.github/workflows/release.yaml@main
    with:
      binary_name: my-cli-tool
      build_args: '--release --features vendored'
      runs-on: |
        [
          {"runs-on": "ubuntu-24.04", "target": "x86_64-unknown-linux-musl"},
          {"runs-on": "windows-latest", "target": "x86_64-pc-windows-msvc"}
        ]
```

### Inputs for `release.yaml`

- `binary_name` (required): Name of the binary/executable.
- `build_args` (optional, default: `--release`): Flags for `cargo build`.
- `runs-on` (optional, default includes multiple platforms): JSON array of runner/target entries.
- `setup-runs-on` (optional, default: `ubuntu-latest`): Runner used by the setup and draft-release jobs.

## Usage: Publish Workflow

The `publish.yaml` workflow publishes a crate to crates.io. Example usage:

```yaml
name: Publish Crate

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  publish:
    uses: unbounded-tech/workflows-rust/.github/workflows/publish.yaml@main
    secrets:
      crates_io_token: ${{ secrets.CRATES_IO_TOKEN }}
    with:
      manifest_path: Cargo.toml
      cargo_publish_args: '--locked'
```

### Inputs for `publish.yaml`

- `manifest_path` (optional, default: `Cargo.toml`): Path to the crate manifest to publish.
- `cargo_publish_args` (optional, default: `--locked`): Additional arguments for `cargo publish`.
- `dry_run` (optional, default: `false`): If `true`, runs `cargo publish --dry-run`.
- `runs-on` (optional, default: `ubuntu-latest`): Runner label used for the job.

### Secrets for `publish.yaml`

- `crates_io_token` (required): crates.io API token.

## Extending

- Customize release matrix targets via `release.yaml` `runs-on` input.
- Adjust cargo arguments to suit workspace layouts or feature flags.
- Chain `publish.yaml` after your version-tag workflow to publish only tagged releases.

## License

MIT
