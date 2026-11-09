# env-example-sync-cli

Scan code for environment variables, compare with `.env.example`, and fail CI when they drift.

[![CI](https://github.com/haiderstudio/env-example-sync-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/haiderstudio/env-example-sync-cli/actions/workflows/ci.yml) [![Licence: MIT](https://img.shields.io/badge/Licence-MIT-yellow.svg)](LICENSE)

## What it does

Every project has a moment where someone adds `os.environ["NEW_KEY"]` to code and forgets `.env.example`, and the next developer's setup breaks silently. `env-example-sync` catches that before it ships:

- **Missing** — used in code, but not documented in `.env.example`.
- **Extra** — documented in `.env.example`, but no longer used in code.
- **Exit code 1 on drift, 0 when in sync** — so it works as a CI gate.
- **`--fix`** appends missing variables to `.env.example` as `NAME=` — no value is ever invented, and extra variables are never deleted automatically.

It scans Python, JavaScript/TypeScript, Go, PHP, and shell files, and skips `node_modules`, `.git`, virtualenvs, and `.env*` files themselves.

## What it does NOT do

- It does not read, print, or upload any real secret value. It only ever reports variable **names**.
- It does not load a real `.env` file, and it never touches one.
- It does not delete variables from `.env.example` — removing a documented variable is a human decision, not an automated one.
- It does not parse code as an AST. It matches text patterns, so a variable named only in a comment still counts as used. That is deliberate: it errs toward keeping a variable documented rather than deleting it.
- Shell scripts only match uppercase names (`$HOME` counts, `$home` does not), because lowercase shell words are locals by convention, not env vars.
- It sends nothing anywhere. Everything runs locally, Python standard library only.

## Install

```sh
pip install env-example-sync-cli
```

Or clone from GitHub instead (Python 3.10+ is enough):

```sh
git clone https://github.com/haiderstudio/env-example-sync-cli.git
cd env-example-sync-cli
python3 src/env_example_sync.py --help
```

## Usage

From your project root, where `.env.example` lives:

```sh
python3 src/env_example_sync.py .
```

Real output on the bundled sample project (see `examples/sample-project`, where `TIMEOUT` is used but undocumented and `OLD_VAR` is documented but unused):

```text
Checked: 1 file(s)
Used in code: 3
Documented in .env.example: 3
Missing from .env.example (used but not documented):
  - TIMEOUT
Extra in .env.example (documented but not used):
  - OLD_VAR
Result: DRIFT
```

That run exits with code 1 — exactly what a CI gate wants.

Other useful forms:

```sh
python3 src/env_example_sync.py --env-file examples/sample-project/.env.example examples/sample-project
python3 src/env_example_sync.py --fix examples/sample-project
```

## Example

See `examples/` for a runnable walkthrough: it creates a throwaway project with a `.env.example` listing `OLD_VAR` and code using `TIMEOUT`, runs the tool, and shows only those two variables reported. The same scenario is covered by the integration test, so the example and the tests cannot drift apart.

## How it is tested

```sh
python3 tests/test_env_example_sync.py
```

Expected output: `All 3 tests passed.` The suite covers the missing/extra split (`TIMEOUT` reported missing, `OLD_VAR` reported extra), the `--fix` behaviour (appending missing variables as blank entries and never deleting extras), and the fact that nothing is written when the files are already in sync. Tests run on the exact files in this repo, with no network and no third-party packages.

## Contributing

Small, focused improvements are welcome. Open an issue first for anything larger than a typo, keep the safety rules above intact, and run the test suite before opening a pull request. See `CONTRIBUTING.md`.

## Licence

MIT — see `LICENSE`. Original work, written for this repo; it builds on git's own `--merged` and `-vv` output, not on any other cleanup tool's code.
