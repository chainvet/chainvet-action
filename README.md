# ChainVet GitHub Action

Scan Solidity smart contracts with [ChainVet](https://github.com/chainvet/chainvet)
in CI and surface findings in GitHub code scanning (the **Security** tab) via SARIF.

## Usage

```yaml
name: ChainVet
on: [push, pull_request]

permissions:
  security-events: write   # required to upload SARIF
  contents: read

jobs:
  chainvet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
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
| `sarif-file`   | `chainvet.sarif` | Where to write the SARIF report. |
| `upload-sarif` | `true`           | Upload the SARIF to GitHub code scanning. |
| `version`      | `main`           | ChainVet git ref to install `chainvet-ci` from. |

The action installs `chainvet-ci` from source (Z3 + Rust toolchain), runs the scan,
uploads the SARIF even when the scan fails the threshold, and then exits non-zero
if `fail-on` was met — so the SARIF always lands in code scanning while still gating
the job.

> This action is a thin wrapper over the `chainvet-ci` frontend. For local use,
> `cargo install --git https://github.com/chainvet/chainvet chainvet-ci` and run
> `chainvet-ci <path> --sarif out.sarif`.
