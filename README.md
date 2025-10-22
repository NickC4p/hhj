
![HHJLogo](https://github.com/NickC4p/hhj/blob/main/%20%20hhj-github-orizzontal.png)
# 🧩 HHJ Engine

[![GitHub release](https://img.shields.io/github/v/release/NickC4p/hhj.svg?sort=semver)](https://github.com/NickC4p/hhj/releases/latest)
[![License: MIT](https://img.shields.io/badge/license-DIO-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.8%2B-yellow.svg)](https://www.python.org/)


> hhj logo ©2025 NickC4p — all rights reserved

HHJ Engine (Hyper Handling Junction) is a small declarative orchestration tool written in Python. It reads `.hhj` YAML files and automates environment setup, dependency installation, downloads and extraction of files, and workflow execution on macOS, Linux, and Windows.

Table of Contents
- Overview
- Features
- Example `project.hhj`
- CLI Usage
- Configuration reference
- Workflow steps (supported actions)
- Safety & security
- Development & contribution
- Troubleshooting
- License

Overview
--------
HHJ interprets a declarative config (`.hhj`) describing project requirements and executes them in order. It focuses on reproducible, minimal operations and is intentionally conservative — operations are dry-run by default. Use `--apply` to perform real changes.

Features
--------
- Parse YAML configuration using `pyyaml`.
- Manage Python environments with `venv`.
- Install Python packages (`pip`) into venv, user, or optionally global.
- Download remote files using `requests` with optional SHA256 verification.
- Extract archives: `.zip`, `.tar.gz`, `.tgz`, `.tar.bz2`.
- Run commands with `subprocess` (supports timeouts and retries).
- File operations: copy, move, create directories, set permissions.
- Declarative workflows with ordered steps.
- Dry-run mode by default; explicit `--apply` to perform operations.

Example `project.hhj`
---------------------
```yaml
project:
  name: "MyApp"
  version: "1.0"
  description: "Example demo"

dependencies:
  venv: true
  pip:
    - requests>=2.30
    - pyyaml>=6.0
  system:
    - git

files:
  download:
    - url: "https://files.pythonhosted.org/packages/source/r/requests/requests-2.31.0.tar.gz"
      sha256: "PLACE_REAL_SHA256"
      extract_to: "vendor"

workflow:
  - say: "Initializing environment"
  - create_venv: true
  - run: "python -m pip install -r requirements.txt"
    timeout: 300
    retries: 2
  - run: "python main.py"
  - say: "Setup complete!"
```

CLI Usage
---------
Basic commands:
- Dry-run (default):
  python hhj_engine.py
- Apply changes:
  python hhj_engine.py --apply
- More info / debug:
  python hhj_engine.py --verbose
- Allow global installs (dangerous):
  python hhj_engine.py --force-global

Specify a different config file:
  python hhj_engine.py --config setup.hhj

Suggested installation (if you don't have dependencies):
- Create and activate a venv (macOS/Linux):
  python3 -m venv .venv
  source .venv/bin/activate
- Windows (PowerShell):
  python -m venv .venv
  .\.venv\Scripts\Activate.ps1

Install runtime deps:
  pip install -r requirements.txt
or
  pip install requests pyyaml

Configuration reference
-----------------------
Top-level keys:
- project: metadata (name, version, description)
- dependencies:
  - venv: true/false
  - pip: list of pip specifiers
  - system: list of system packages to attempt to install (platform-specific)
- files:
  - download: list of objects {url, sha256 (optional), extract_to (optional), dest (optional)}
- workflow: ordered list of steps (see supported actions)

Supported workflow steps
------------------------
Each workflow entry is an object with a single action key:

- say: string
  - Prints a message to the console.
- create_venv: true|path
  - Create a Python venv (default path: .hhj-venv).
- run: string or object
  - Run a shell command.
  - Optional fields: timeout (seconds), retries (int), env (dict).
  Example:
    - run:
        cmd: "python -m pip install -r requirements.txt"
        timeout: 300
        retries: 2
- copy: {src: path, dest: path}
- move: {src: path, dest: path}
- download: {url: string, sha256: string (optional), dest: path (optional), extract: bool}
- extract: {archive: path, dest: path}
- remove: path

Notes:
- Steps run sequentially.
- Steps should be idempotent where possible to allow reruns.
- Dry-run prints operations without executing them.

Platform-specific behavior
--------------------------
System package installation is attempted using the detected package manager:
- macOS: Homebrew (brew)
- Debian/Ubuntu: apt (sudo may be required)
- Windows: Chocolatey (choco)

HHJ will not perform privileged operations without explicit permission flags (e.g., `--force-global`). Adjust system installs manually if required.

Safety & security
-----------------
- Dry-run by default: safe inspection before making changes.
- Downloads can include `sha256` to verify integrity; you should always provide checksums for remote artifacts.
- Avoid `--force-global` unless you understand the system-wide impact.
- Commands run via subprocess — avoid running untrusted workflow files.

Logging
-------
Console output is prefixed with `[HHJ]` and supports info, warning, and error messages. Use `--verbose` to see debug-level details.

Development & Contributing
--------------------------
- Run tests (if present) using pytest:
  pytest
- Formatting & linting: consider using black/flake8.
- Contributions welcome: open an issue or a pull request with a clear description of the change.
- If you'd like, I can open a PR updating this README and include CI examples.

Troubleshooting
---------------
- If venv creation fails, check Python version (3.8+ recommended) and permissions.
- If downloads fail, check network and URL; re-run with `--verbose` for detailed HTTP errors.
- On Windows, prefer PowerShell activation or adjust commands to use appropriate path separators.

License
-------
MIT License — © 2025 HHJ Engine Authors.

Authors & Contact
-----------------
- Maintainer: NickC4p
- For issues and PRs: https://github.com/NickC4p/hhj/issues

Acknowledgements
----------------
- Built with pyyaml, requests, and packaged-in standard library modules.

