# Git Repository Update

CustomTkinter app for scanning local Git repositories and updating them in bulk.

## What It Does

- Scans configured roots for Git repositories
- Records repository status in `repository_manifest.json`
- Detects SSH key configuration where possible
- Runs `git pull --ff-only` across selected repositories
- Supports dry-run workflows

This standalone repository vendors the small `packages_custom/` helper module used by the app. The vendored copy is `packages_custom` version `0.1.0` from source commit `3e87d89`. Once published, the source repository URL will be <https://github.com/Cure-Interactive/packages_custom>.

## Requirements

- Python 3.10+
- Git installed and available on `PATH`
- Dependencies from `requirements.txt`

## Install

```bash
python setup.py --venv
```

Or manually:

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

On Linux or macOS, activate the virtual environment with `source .venv/bin/activate`.

## Configure And Run

```bash
python git_repository_update.py
```

On first run, `config.json` is created from `config_default.json`. Add scan roots in the Config tab, run Refresh, then run Pull when the planned repository list looks correct.

See `wiki.md` for detailed workflow notes.
