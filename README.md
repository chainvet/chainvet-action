# Chainvet GitHub Action

Scan Solidity smart contracts with [Chainvet](https://github.com/chainvet/chainvet)
in CI and surface findings in GitHub code scanning (the **Security** tab) via SARIF.

## Usage

```yaml
name: Chainvet
on: [push, pull_request]

permissions:
  security-events: write   # required to upload SARIF
  contents: read

jobs:
  chainvet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: chainvet/chainvet-action@v1
        with:
          path: contracts
          mode: hybrid
          fail-on: high
```

## Inputs

| Input          | Default          | Description |
|----------------|------------------|-------------|
| `path`         | `.`              | File or directory to scan. |
| `mode`         | `hybrid`         | `static` \| `symbolic` \| `fuzzing` \| `hybrid`. |
| `fail-on`      | `high`           | Fail the job when a finding meets this severity (`high`/`medium`/`low`/`none`). |
| `fail-on-confidence` | `candidate` | Only fail on findings at this confidence tier or above (`candidate`/`confirmed`). `confirmed` gates only on findings corroborated by symbolic/fuzz execution. |
| `sarif-file`   | `chainvet.sarif` | Where to write the SARIF report. |
| `upload-sarif` | `true`           | Upload the SARIF to GitHub code scanning. |
| `version`      | `latest`         | Chainvet release to use — a tag like `v0.2.0`, or `latest`. |

The action installs `chainvet-ci` by **downloading the prebuilt binary** for the
selected release (seconds), falling back to a **from-source build** (Z3 + Rust) if
no matching prebuilt exists — e.g. a branch name in `version`, or a non-x86_64-Linux
runner. It runs the scan, uploads the SARIF even when the scan fails the threshold,
and then exits non-zero if `fail-on` was met — so the SARIF always lands in code
scanning while still gating the job.

> This action is a thin wrapper over the `chainvet-ci` frontend. For local use,
> `cargo install --git https://github.com/chainvet/chainvet chainvet-ci` and run
> `chainvet-ci <path> --sarif out.sarif`.
