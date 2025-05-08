
# workflows-rust

Reusable GitHub Actions workflows for Rust CLI quality checks and release automation.

## Overview

This repository provides workflows to automate quality assurance and release processes for Rust CLI tools. It includes two callable workflows: `quality.yaml` for building and testing, and `release.yaml` for building and publishing cross-platform binaries. Additionally, it includes workflows for versioning, tagging, and creating GitHub Releases.

### Workflows Included

- **quality.yaml**  
  _Trigger:_ `workflow_call`  
  _Purpose:_ Builds and tests a Rust project across a matrix of platforms, with customizable build and test arguments.  
  _Typical Use:_ Ensures code quality before release by running builds and tests.

- **release.yaml**  
  _Trigger:_ `workflow_call`  
  _Purpose:_ Builds Rust binaries for a matrix of platforms and publishes them to a GitHub Release.  
  _Typical Use:_ Automates cross-platform binary builds and release artifact publishing.

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
    uses: harmony-labs/workflows-rust/.github/workflows/quality.yaml@main
    with:
      cargo_build_args: '--no-default-features --verbose'
      cargo_test_args: '--no-default-features --verbose'
      cargo_incremental: false
      platforms: '[{"runs-on": "ubuntu-latest"}, {"runs-on": "macos-latest"}]'
      verbose_logging: true
```

### Inputs for `quality.yaml`

- `cargo_build_args` (optional, default: `--no-default-features --verbose`): Additional arguments for `cargo build`.
- `cargo_test_args` (optional, default: `--no-default-features --verbose`): Additional arguments for `cargo test`.
- `cargo_incremental` (optional, default: `false`): Enable incremental compilation.
- `platforms` (optional, default: `[{"runs-on": "ubuntu-latest"}]`): JSON array of platforms for the build matrix.
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
    uses: harmony-labs/workflows-rust/.github/workflows/release.yaml@main
    with:
      binary_name: my-cli-tool
      build_args: '--release --features vendored'
      platforms: |
        [
          {"os-name": "Linux-x86_64", "runs-on": "ubuntu-24.04", "target": "x86_64-unknown-linux-musl"},
          {"os-name": "Windows-x86_64", "runs-on": "windows-latest", "target": "x86_64-pc-windows-msvc"}
        ]
```

### Inputs for `release.yaml`

- `binary_name` (required): Name of the binary/executable.
- `build_args` (optional, default: `--release --features vendored`): Flags for `cargo build`.
- `platforms` (optional, default includes multiple platforms): JSON array of platforms for the build matrix.

## Extending

- Customize the `platforms` input to target specific operating systems or architectures.
- Add logic in your repository to update Homebrew formulas or Scoop manifests if needed (not implemented in these workflows).
- Modify build/test arguments to suit your project's needs.

## License

MIT
