# Skill Development Guide

This guide defines the conventions for building PSPM skills in this repository.

## Folder Structure

Every skill lives under `skills/<skill-name>/` and follows this layout:

```
skills/<skill-name>/
├── SKILL.md              # Skill documentation (usage only)
├── pspm.json             # PSPM manifest (name, version, metadata)
└── runtime/              # Isolated execution environment
    ├── pyproject.toml    # Python: deps + tool config (uv)
    ├── uv.lock           # Python: pinned dependency versions
    ├── .python-version   # Python: version pin
    ├── package.json      # TypeScript: deps + scripts (bun)
    ├── bun.lock          # TypeScript: pinned dependency versions
    └── <script files>    # Flat in runtime/, no nested scripts/ folder
```

The top level of each skill holds **standalone skill files** (SKILL.md, pspm.json, config).
The `runtime/` folder is a **self-contained project** with its own dependencies — fully isolated per skill.

## Runtime Environments

### Python Skills — use `uv`

Each Python skill's `runtime/` is a uv project.

**Setup:**
```bash
cd skills/<skill-name>/runtime
uv init --name <skill-name>
uv add <runtime-deps>            # e.g. yt-dlp
uv add --dev ruff pyright        # dev tooling
```

**Execution (from the skill folder):**
```bash
uv run --project runtime runtime/<script>.py [args]
```

**Dev checks (from runtime/):**
```bash
uv run ruff check .
uv run ruff format .
uv run python -m pyright .
```

**pyproject.toml conventions:**
```toml
[project]
requires-python = ">=3.10"

[tool.ruff]
target-version = "py310"
line-length = 120

[tool.ruff.lint]
select = ["E", "W", "F", "I", "UP", "B", "SIM", "RUF"]

[tool.pyright]
pythonVersion = "3.10"
include = ["."]
typeCheckingMode = "standard"
```

### TypeScript Skills — use `bun`

Each TypeScript skill's `runtime/` is a bun project.

**Setup:**
```bash
cd skills/<skill-name>/runtime
bun init
bun add <runtime-deps>
```

**Execution (from the skill folder):**
```bash
bun run runtime/<script>.ts [args]
```

## SKILL.md Conventions

SKILL.md should only document **how to use** the skill, not how to develop it:

- Brief description of what the skill does
- Prerequisites (external tools needed)
- Usage examples with `uv run --project runtime` or `bun run runtime/`
- Options/flags reference
- Limitations

## pspm.json

Every skill has a `pspm.json` manifest for PSPM registry compatibility:

```json
{
  "name": "@user/<username>/<skill-name>",
  "version": "1.0.0",
  "description": "Brief description"
}
```

## Output Directory

Skills that produce output files should default to an `output/` directory inside the skill folder (resolved from the script's location, not cwd). The script should auto-create it.

`output/` is globally gitignored.

## .gitignore

The repo-level `.gitignore` covers all skills:

```
.venv/
.ruff_cache/
__pycache__/
*.pyc
output/
node_modules/
bun.lockb
```

Skill-specific ignores go in `skills/<skill-name>/.gitignore`.

## Adding a New Skill

1. Create the folder: `mkdir -p skills/<skill-name>/runtime`
2. Add `SKILL.md` at the skill root
3. Add `pspm.json` at the skill root
4. Initialize the runtime:
   - Python: `cd skills/<skill-name>/runtime && uv init --name <skill-name>`
   - TypeScript: `cd skills/<skill-name>/runtime && bun init`
5. Add dependencies and write scripts in `runtime/`
6. Test execution from the skill folder using `--project` flag
