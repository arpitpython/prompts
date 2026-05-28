Act as an expert Python DevOps and packaging engineer. Your task is to generate a complete, production-ready Windows EXE packaging setup for this Python desktop automation application using PyInstaller.

---

## Step 1 — Analyse This Project First

Before writing anything, read and understand:

1. The **entry point** (e.g. `main.py`, `app.py`) — confirm the exact filename
2. Every **import** used across all `.py` files — identify UI library, automation libs, data libs
3. Any **data files, assets, or config files** that must be bundled (icons, CSVs, JSON configs, SSL certs, etc.)
4. Any **Playwright or Selenium** usage — check if browsers need bundling
5. The **project name** — use it for EXE filename, window title, and spec file name

Grep the codebase thoroughly before proceeding. Do not assume — verify every dependency.

---

## Step 2 — Deliver the Following Files

### 1. `build.bat` — Developer Build Script

Requirements:
- Cleans `build/` and `__pycache__` before every run
- **ONLY deletes the EXE itself from `dist/`** — never wipes the entire `dist/` folder; other files there (user presets, config) must survive across builds
- Activates the virtual environment (`venv\Scripts\activate`) if it exists
- Installs / updates all dependencies from `requirements.txt` automatically
- Installs Playwright browsers if Playwright is detected in the project
- Runs PyInstaller using the `.spec` file (not CLI flags — spec file only)
- Prints clear success/failure messages
- Opens the `dist/` folder on success so the EXE is immediately visible
- Saves the `.bat` to the project root

### 2. `<project_name>.spec` — Production PyInstaller Spec File

Requirements:
- `onefile=True`, `console=False` (windowed, no terminal)
- `hiddenimports` — include every library detected in the project PLUS the standard hidden imports for:
  - pandas, numpy, openpyxl (all sub-modules that commonly break)
  - requests, urllib3, certifi (SSL support)
  - Playwright and/or Selenium if used
  - The UI library detected (ttkbootstrap, Flet, customtkinter, tkinter, etc.)
  - Any other detected dependency
- `datas` — bundle:
  - SSL certificates from certifi (`certifi.where()`)
  - Asset files found in the project (icons, templates, etc.)
  - **Never bundle** any user-editable config: `.json`, `.xlsx`, `.ini`, `.yaml`, `.csv` config files — not even as a "default seed". They must live next to the EXE, managed by the user.
  - Playwright browser binaries if used
- `binaries` — include any required DLLs detected
- `excludes` — exclude test frameworks, dev tools, and anything not needed at runtime to keep EXE lean
- Version metadata block: `name`, `version='1.0.0'`, `description` filled from project context
- Icon: use `icon='icon.ico'` if an `.ico` file exists in the project; otherwise omit (never use a placeholder path)
- UPX compression disabled (`upx=False`) for Windows compatibility

### 3. Path Helper Snippet

Include a ready-to-paste Python snippet for resolving paths in frozen EXEs:
- Files bundled inside the EXE (e.g. certifi certs): use `sys._MEIPASS`
- User-editable config files (mapping xlsx, JSON, INI): use `Path(sys.executable).parent` — next to the EXE, never inside it
Explain exactly which files in this project use each pattern.

### 4. Encoding Fix

If any file I/O or string processing is detected, add the encoding safety fix (`PYTHONUTF8=1` env var + `encoding='utf-8'` guidance).

### 5. Post-Build Checklist

A short checklist the developer runs after the EXE is built:
- [ ] EXE launches on a clean machine (no Python installed)
- [ ] SSL / HTTPS calls succeed
- [ ] File read/write in user directory works
- [ ] All UI components render correctly
- [ ] Any automation (Playwright/Selenium) runs without errors

---

## Quality Rules

- **No placeholders** — every path, import, and filename must be real values derived from this project
- **No guessing** — if a dependency is unclear, grep for it first
- **Production grade** — this EXE will be distributed to non-technical end users; it must work first run
- **Complete files only** — deliver the full `.bat` and `.spec` content, not partial snippets
- **Explain any non-obvious decision** in one line inline

Output the `.bat` and `.spec` as complete, ready-to-save code blocks. Then write the path helper snippet and post-build checklist.

---

## Universal Rules — User-Editable Config Files

### Never bundle config files — never seed them on first run

User-editable files (JSON, XLSX, INI, YAML, CSV config) must **never** be added to `datas` in the spec file unless the user explicitly requests it.

- Do **not** include any `.json`, `.xlsx`, `.ini`, `.yaml`, or `.csv` config file in `datas`
- Do **not** copy or seed any config file from `sys._MEIPASS` on first run
- If a config file does not exist next to the EXE on first launch, the app must handle it gracefully — prompt the user, show a setup screen, or use safe in-memory defaults
- If a config file already exists next to the EXE, a new build must never overwrite or replace it
- **Why:** Users accumulate real data in these files over time (presets, mappings, credentials). Auto-seeding from a bundled default risks overwriting live data on upgrade, and bundling a stale default creates a false sense of correctness.
- **Exception:** Only bundle a config file if the user explicitly says "bundle this file" — document the reason inline in the spec.
