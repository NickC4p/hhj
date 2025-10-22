# HHJ Engine — v1.1b1 (beta1)

HHJ Engine is a single-file orchestrator to run workflows described in a YAML `.hhj` configuration.  
This repository contains hhj_engine.py version 1.1b1 — a beta release with experimental features and known issues.

WARNING: This is a beta release and intentionally contains unstable/experimental behavior. Read the Known Bugs section carefully before using it on important systems.

---

## What’s new in 1.1b1 (beta1)

- Experimental parallel downloads (configurable via `behavior.parallel_downloads`)
- Smart dependency resolver for pip dependencies (experimental and naïve)
- CLI colorized output, with `--color` / `--no-color` flags and automatic TTY detection
- Numerous intentionally included unwanted/known bugs for testing and QA

---

## Known Bugs (intentional)

This beta intentionally contains these issues (and some additional quirks):

- Random download failures — especially when `parallel_downloads` is enabled (race/overwrite in parallel mode)
- Incorrect dependency update logs — the smart resolver may claim updates that didn't take place or pick wrong versions
- Occasional CLI output glitch on macOS — intermittent or malformed color escape sequences
- SHA256 verification may occasionally use a shortened/prefix check (incorrect)
- Random truncation of command stdout in some cases
- Random exit code override on failure (may exit 0 despite a fatal error)
- Edge-case archive extraction and suffix detection may mis-detect certain filenames

If you need a stable engine, do not use this beta on production systems. Use it for testing, QA, or to reproduce flaky behavior.

---

## Requirements

- Python 3.8+
- pip packages:
  - pyyaml
  - requests

Install dependencies:

```bash
pip install pyyaml requests
```

---

## Usage

Default behavior is dry-run. Use `--apply` to actually perform the actions.

Basic invocation:

```bash
python hhj_engine.py --config project.hhj
```

Common CLI flags:

- `--config, -c` Path to the `.hhj` YAML config (default: `project.hhj`)
- `--apply` Apply changes (default is dry-run)
- `--force-global` Allow pip global installs (dangerous; required to run global installs)
- `--verbose, -v` Verbose debug output
- `--workdir, -w` Working directory (project root)
- `--retries, -r` Retries for network operations (default: 2)
- `--color` Force colorized output
- `--no-color` Disable colorized output

Example:

```bash
# Dry run (default)
python hhj_engine.py --config project.hhj

# Apply changes and force global pip installs (dangerous)
python hhj_engine.py --config project.hhj --apply --force-global --color -v
```

---

## Configuration (project.hhj) — highlights

hhj_engine expects a YAML file describing environment, dependencies, files, and a workflow. Only the relevant parts for the new features are shown here.

Example with experimental parallel downloads and pip dependencies:

```yaml
project:
  name: sample

behavior:
  # Toggle experimental parallel downloads
  parallel_downloads: true

environment:
  MY_VAR: "value"

dependencies:
  venv: true
  pip:
    - "requests==2.31.0"
    - spec: "somepackage>=1.2.0"   # dict form also supported
  system:
    - git
    - curl

files:
  download:
    - url: "https://example.com/files/tool-1.2.3.tar.gz"
      sha256: "0123456789abcdef..."     # optional
      extract_to: "tools/tool-1.2.3"
      install_to: "/usr/local/bin"      # used only to decide pip/global in some cases
    - url: "https://example.com/files/asset.zip"
      extract_to: "assets"
```

Notes:
- `behavior.parallel_downloads: true` enables the experimental parallel downloader (may be unstable).
- `dependencies.pip` accepts either simple strings or dicts with `spec` keys. The smart resolver will deduplicate and (naïvely) pick versions.
- The engine will create a virtualenv if `dependencies.venv: true` or if workflow steps request a venv.

---

## How the experimental features behave

Parallel downloads:
- Uses a thread pool to download multiple files concurrently.
- Filename derivation is intentionally simple and non-atomic — collisions and overwrites can occur in this beta.

Smart dependency resolver:
- Naïve deduplication and version selection (uses lexicographic comparison).
- May mis-handle semantic versioning, and logs may state updates even when nothing changed.

Colorized CLI:
- Colors are on by default when stdout is a TTY.
- Use `--color` / `--no-color` to override. Small chance of malformed escapes on macOS in this beta.

---

## Troubleshooting & tips

- If downloads randomly fail with parallel mode, disable it by setting `behavior.parallel_downloads: false` (or remove the key).
- If a pip install fails in `venv` mode, check that the virtualenv was created and that `pip` exists inside it. Use `--apply` to create the venv (dry-run will not create it).
- For systems where colored output appears broken (macOS or certain terminals), run with `--no-color`.
- The smart resolver may produce wrong versions — inspect the resolved list by running with `--verbose`.
- Keep a backup of any important environments — this beta intentionally behaves unpredictably.

---

## Development / Contributing

This beta deliberately includes unwanted bugs, so if you're contributing fixes please:

- Create an issue for each bug you intend to fix and reference `1.1b1`.
- Add tests that reproduce the flaky behavior before fixing it.
- Prefer deterministic atomic file writes for downloads (download to a unique temp file then rename).
- Replace naïve version comparisons with `packaging.version.parse()` when improving the resolver.

---

## Example workflow snippets

Run a shell command step:

```yaml
workflow:
  - run: "echo 'Hello world' && python --version"
```

Create a venv and install requirements:

```yaml
workflow:
  - create_venv: true
    path: ".venv"
  - install_requirements: true
    file: requirements.txt
    target: venv
```

---

## License

This example file does not specify a license. Add a `LICENSE` file to the repository if you intend to change the license terms.

---

## A final word

This README documents hhj_engine v1.1b1 (beta1). I intentionally preserved several unwanted bugs in the code to allow testing and QA workflows that need flaky behavior. Use with care and always test in an isolated environment first.
