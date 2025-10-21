# 🧩 HHJ Engine

[![GitHub release](https://img.shields.io/github/v/release/NickC4p/hhj.svg?sort=semver)](https://github.com/NickC4p/hhj/releases/latest)

![HHJLogo](https://github.com/NickC4p/hhj/blob/main/Images/J.png)
> hhj logo ©2025 NickC4p all rights reserved

**HHJ Engine** (Hyper Handling Junction) is a Python-based declarative orchestration engine.  
It reads `.hhj` YAML files and automates environment setup, Python and system dependency installation, file downloads, extraction, and workflow execution on macOS, Linux, and Windows.

---

## Overview

HHJ Engine interprets a declarative configuration (`.hhj`) describing project requirements.  
It executes the following real operations:

- **Parse YAML** using `pyyaml`.
- **Manage Python environments** using `venv` (creates virtual environments automatically).
- **Install Python packages** with `pip` in `venv`, `user`, or `global` scope.
- **Download remote files** using `requests` with optional SHA256 verification.
- **Extract archives** (`.zip`, `.tar.gz`, `.tgz`, `.tar.bz2`) with `zipfile` and `tarfile`.
- **Run commands** safely via `subprocess`, supporting timeouts, retries, and environment injection.
- **Move or copy files**, create directories, and manage permissions with `os` and `shutil`.
- **Workflow orchestration**: executes YAML-defined steps in order (`say`, `run`, `create_venv`, `copy`, `move`, etc.).
- **Dry-run mode** by default; `--apply` enables real operations.

---

## Example `project.hhj`

```yaml
project:
  name: "MyApp"
  version: "1.0"

dependencies:
  venv: true
  pip:
    - requests>=2.30
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
  - run: "python -m pip install requests"
  - run: "python main.py"
  - say: "Setup complete!"
```

---

## Usage

```bash
python hhj_engine.py                 # dry-run (default)
python hhj_engine.py --apply         # perform actions
python hhj_engine.py --verbose       # debug output
python hhj_engine.py --force-global  # allow global installs
```

Specify a different config file:

```bash
python hhj_engine.py --config setup.hhj
```

---

## How It Works

1. **Load Configuration**: parses `.hhj` YAML with `pyyaml`.
2. **Environment Setup**: creates virtualenv if `venv: true`.
3. **Dependency Installation**:
   - Python packages via `pip`
   - System packages via Homebrew (macOS), apt (Linux), or Chocolatey (Windows)
4. **File Management**: downloads files, verifies SHA256, extracts archives, moves files.
5. **Workflow Execution**: executes steps sequentially with environment variables, retries, and optional timeout.
6. **Cleanup**: removes temporary directories and files automatically.

All operations respect `--dry-run` unless `--apply` is specified.

---

## Technical Details

- **Python Modules Used**: `pyyaml`, `requests`, `os`, `shutil`, `subprocess`, `zipfile`, `tarfile`, `hashlib`, `tempfile`, `sys`.
- **Cross-platform support**: macOS, Linux, Windows.
- **Safety Features**:
  - Dry-run by default
  - Explicit flags for global changes
  - Optional checksum verification for downloads
- **Logging**: console output prefixed with `[HHJ]` including info, warnings, and errors.

---

## License

MIT License — © 2025 HHJ Engine Authors.

### How to install
> if you don't have pyyaml or requests keep reading

If you have installed python from **Homebrew** you will need these commands
```bash
 python3 -m venv path/to/venv
    source path/to/venv/bin/activate
```
and then
```bash
pip install requests pyyaml
