# Repository Bulk GUI — wiki.md

A **CustomTkinter** GUI that replaces your `repository_bulk_refresh.py` + `repository_bulk_pull.py` workflow in one app:

- **Refresh**: scan for git repos, detect SSH keys, write `config/repository_bulk_list.json`
- **Pull**: read that list, apply filters, run `git pull --ff-only` per repo (optionally with per-repo SSH key)

---

## 0) Folder layout (expected)

Put the GUI script **in the same folder as the existing tools**, with `config/` beside it:

```

repository_bulk/
repository_bulk_gui.py
requirements.txt
install.py                  (optional helper installer)
icon.ico                    (optional)
icon.png                    (optional)
repository_bulk_pull.py     (optional legacy)
repository_bulk_refresh.py  (optional legacy)
config/
repository_bulk_pull.json
repository_bulk_refresh.json
repository_bulk_list.json          (generated)
default/
repository_bulk_pull.json
repository_bulk_refresh.json

````

---

## 1) Minimum install (simplest)

### Requirements
- **Python 3.10+** (3.12 is fine)
- `pip`
- **Git** installed and available on PATH (`git --version`)
- `requirements.txt` contains:
  - `customtkinter>=5.2.2`

### Install
From the `repository_bulk/` folder:

**Windows (PowerShell)**
```powershell
py -m pip install -r requirements.txt
````

**Linux/macOS**

```bash
python3 -m pip install -r requirements.txt
```

---

## 2) Recommended install (venv + local installer script)

If you’re keeping an `install.py` next to `requirements.txt`, use it to install consistently **relative to the script folder**, not your current working directory. 

### Add `install.py`

Use the installer you already have (your uploaded file content is an `install.py`-style script). 
Place it in this folder and name it **`install.py`**:

```
repository_bulk/install.py
repository_bulk/requirements.txt
```

### Run it (creates `.venv` and installs requirements)

**Windows**

```powershell
py install.py --venv
```

**Linux/macOS**

```bash
python3 install.py --venv
```

### Activate the venv (optional)

**Windows**

```powershell
.\.venv\Scripts\activate
```

**Linux/macOS**

```bash
source ./.venv/bin/activate
```

---

## 3) Run the app

### Basic run

**Windows**

```powershell
py repository_bulk_gui.py
```

**If you used venv**

```powershell
.\.venv\Scripts\python.exe repository_bulk_gui.py
```

**Linux/macOS**

```bash
python3 repository_bulk_gui.py
```

---

## 4) First run behavior (defaults)

The app expects these config files:

* `config/repository_bulk_refresh.json`
* `config/repository_bulk_pull.json`

If they’re missing, use:

* **Config tab → “Bootstrap Defaults (non-destructive)”**

This copies from:

* `config/default/repository_bulk_refresh.json`
* `config/default/repository_bulk_pull.json`

---

## 5) Using the app (minimum steps)

### A) Refresh (scan + write list)

1. Open **Config** tab
2. Add at least one folder to **scan_roots**

   * Example: `D:\Repos`
3. (Optional) set **default_ssh_key**
4. Go to **Operations** tab
5. Click **Refresh (Scan + Write List)**

Result:

* Writes/updates `config/repository_bulk_list.json`

### B) Pull (git pull --ff-only)

1. Click **Pull (git pull --ff-only)**
2. Watch the table status per repo:

   * `Updated`, `No changes`, `Error`, etc.

---

## 6) Using the app (most detailed)

### 6.1 Operations tab controls

#### Buttons

* **Refresh (Scan + Write List)**

  * Walks each `scan_roots` folder
  * Detects git repos (folders that contain `.git`)
  * Detects SSH key per repo (best effort)
  * Writes the repo list JSON

* **Pull (git pull --ff-only)**

  * Loads `config/repository_bulk_list.json`
  * Applies pull filters (allowed/disallowed)
  * Runs `git pull --ff-only` per repo

#### Toggles

* **Dry Run**

  * Refresh: shows what it *would* write
  * Pull: shows what it *would* pull (no git commands executed)

* **Auto-save config before run**

  * Saves Config tab changes automatically before executing operations

#### Repo table

Columns:

* **Repository**: repo path
* **SSH Key**: key file path found/used
* **Status**: queued / found / updated / error / etc.

#### Log box

* Timestamped lines with emoji tags
* Useful for debugging key detection and git failures

---

## 6.2 Config tab fields (what they mean)

### Refresh config (`config/repository_bulk_refresh.json`)

* **scan_roots** (list)

  * Root folders to search for repositories
  * Can be absolute or relative (relative resolves from script folder)

* **ignore_prefixes** (list)

  * Directory name prefixes to skip while walking
  * Typical examples: `$`, `System Volume Information`

* **default_ssh_key** (string)

  * Used when no key can be detected per-repo
  * Leave empty to run without an explicit key

* **output_json** (string)

  * Where to write repo list
  * Default: `config/repository_bulk_list.json`

### Pull config (`config/repository_bulk_pull.json`)

* **allowed_roots** (list)

  * If non-empty: only repos whose path starts with one of these prefixes are included

* **disallowed_roots** (list)

  * Always excluded if repo path starts with one of these prefixes

### Repo list file (`config/repository_bulk_list.json`)

Format:

```json
{
  "I:/Path/To/RepoA": "I:/Path/To/Key.ppk",
  "I:/Path/To/RepoB": ""
}
```

---

## 7) SSH key behavior (practical)

During **Pull**, the app tries to use the repo’s SSH key by setting:

* `GIT_SSH_COMMAND = plink -i "<key>"`

So on Windows, you generally want:

* **PuTTY / Plink installed** and `plink.exe` available on PATH

If you don’t use PuTTY keys / plink:

* leave keys empty, or
* switch the pull logic to `ssh -i` (if your environment expects OpenSSH keys)

---

## 8) Troubleshooting (common failures)

### “git not found”

* Install Git
* Ensure `git` is in PATH:

  ```powershell
  git --version
  ```

### “plink not found”

* Install PuTTY or provide `plink.exe` in PATH
* Quick check:

  ```powershell
  plink -V
  ```

### “Permission denied (publickey)”

* Wrong key, wrong remote, or server doesn’t accept that key
* Validate the repo remote:

  ```powershell
  git -C "I:\Path\To\Repo" remote -v
  ```
* Validate the key is the one that has access

### Repo has no commits (“unborn HEAD”)

* The tool will **skip** repos where `HEAD` doesn’t exist yet.
* Make an initial commit or fetch properly.

---

## 9) Optional: window icon

If you want an icon in the title bar/taskbar, add:

* `icon.ico` (best for Windows)
* `icon.png` (fallback for Linux/macOS)

Place them next to `repository_bulk_gui.py`:

```
repository_bulk_gui.py
icon.ico
icon.png
```

Then ensure the script calls the icon helper (the same pattern you used in your other GUIs).

---

## 10) Suggested workflow

**Fast daily usage**

1. Open app
2. (Optional) toggle **Dry Run** first to sanity check
3. Run **Pull**
4. Only run **Refresh** when:

   * you added/moved repos, or
   * you changed keys, or
   * you changed scan roots / filters

---

## 11) Notes for repo hygiene

* Keep `config/repository_bulk_list.json` in gitignore if you don’t want per-machine paths committed
* Keep `config/default/*.json` committed as “golden defaults”
