# CLAUDE.md

This is a repository of common PSPM (Prompt Skill Package Manager) skills. PSPM is a package manager for prompt skills used by AI coding agents (Claude Code, Cursor, Windsurf, Gemini CLI, etc.). See the [PSPM CLI](https://github.com/nicepkg/pspm) for more information.

## Repository Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Usage documentation
    ├── pspm.json         # PSPM manifest
    └── runtime/          # Isolated execution environment (uv or bun project)
```

- Each skill is self-contained under `skills/<skill-name>/`
- Standalone skill files (SKILL.md, pspm.json) live at the skill root
- All code and dependencies live in `runtime/` — each skill has its own isolated environment

## Conventions

- **Python skills**: Use `uv` for project management, dependency resolution, and script execution. Never use `pip` directly.
- **TypeScript skills**: Use `bun` for project management and script execution. Never use `npm` directly.
- **Execution**: Always use `uv run --project runtime runtime/<script>.py` or `bun run runtime/<script>.ts` from the skill folder.
- **Output**: Skills that produce files should default to `output/` inside the skill folder (auto-created, gitignored).
- **SKILL.md**: Usage documentation only — no development instructions.

## Development

### Python skill checks (from `skills/<skill-name>/runtime/`)

```bash
uv run ruff check .          # Lint
uv run ruff format .         # Format
uv run python -m pyright .   # Typecheck
```

### Python tool config

All Python skills should include ruff and pyright as dev dependencies with this baseline config in `pyproject.toml`:

```toml
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

## Skill guide

See [docs/skill-development-guide.md](docs/skill-development-guide.md) for the full skill development conventions.
