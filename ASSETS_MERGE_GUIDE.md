## Assets generation and merge conflict guide

This project auto-generates `module/*/assets.py` from images under `assets/*/*` using `dev_tools/button_extract.py`. Manual edits to any generated `assets.py` will be overwritten and are likely to cause merge conflicts with upstream.

### Principles
- **Do not edit generated files**: Never hand-edit `module/*/assets.py`.
- **Keep custom assets separate**: Place feature-specific images in a dedicated module to avoid touching upstream-generated modules.
- **Prefer upstream on conflicts**: If a generated file conflicts, keep upstream’s version, and re-generate your custom module.

### Recommended layout for custom features (e.g., gems_farming)
- Put your images here (per server):
  - `assets/cn/gems_farming/*.png`
  - `assets/en/gems_farming/*.png` (optional)
  - `assets/jp/gems_farming/*.png` (optional)
  - `assets/tw/gems_farming/*.png` (optional)
- Generate a new module file:
  - `module/gems_farming/assets.py` (auto-generated)
- Import in your feature code:
  - `from module.gems_farming.assets import *`

This isolates your additions from upstream’s `module/equipment/assets.py`, etc., minimizing conflicts.

### If you must extend an existing module (overlay option)
- Create `module/<module_name>/assets_local.py` and define only your extra constants with unique prefixes (e.g., `GEMS_*`).
- Import locally where needed:
  - `from module.<module_name>.assets import *`
  - `from module.<module_name>.assets_local import *`

This avoids editing the generated `assets.py` while keeping your additions available.

### Regeneration commands (Windows, uv)
Prerequisites (once per machine):
```powershell
uv --version
uv python install 3.9
uv python pin 3.9
uv venv .venv
.\.venv\Scripts\python -V
uv pip install numpy==1.19.5 opencv-python==4.5.3.56 pillow==8.3.2 imageio==2.27.0 tqdm==4.62.3 rich==11.2.0 pywebio==1.6.2
```

Generate only one module (preferred):
```powershell
set PYTHONPATH=.
.\.venv\Scripts\python -c "import sys; sys.path.insert(0, '.'); from dev_tools.button_extract import ModuleExtractor; ModuleExtractor('gems_farming').write()"
```

Re-generate an existing module (only when intended):
```powershell
set PYTHONPATH=.
.\.venv\Scripts\python -c "import sys; sys.path.insert(0, '.'); from dev_tools.button_extract import ModuleExtractor; ModuleExtractor('equipment').write()"
```

Generate everything (slow; may change many files):
```powershell
set PYTHONPATH=.
.\.venv\Scripts\python dev_tools\button_extract.py
```

### Merge conflict resolution workflow
When merging upstream (e.g., `lme_fork/master`) and `assets.py` conflicts:
1. **Prefer upstream for generated files**:
   - `git checkout --theirs module/<name>/assets.py`
   - Or keep your branch version: `git checkout --ours ...` (not recommended)
2. **Re-generate only your custom module** (e.g., `gems_farming`) using the commands above.
3. **Commit only intended changes**:
   - `git add module/gems_farming/assets.py`
   - `git commit -m "Regenerate module/gems_farming/assets.py after merge"`

### Notes
- The generator picks up files with uppercase basenames and optional overrides: `NAME.png`, `NAME.AREA.png`, `NAME.COLOR.png`, `NAME.BUTTON.png`.
- If a server image is missing, the generator falls back to CN (`assets/cn/...`).
- Keep image resolution at 1280x720 to avoid degraded detection.


