# anyt-skills

A collection of example [PSPM](https://github.com/anyt-io/pspm-cli) skills — prompt skill packages for AI coding agents like Claude Code, Cursor, Windsurf, Gemini CLI, and more.

## What is PSPM?

PSPM (Prompt Skill Package Manager) is an open-source package manager for prompt skills. Skills extend AI coding agents with reusable capabilities such as downloading videos, generating reports, or interacting with APIs. See the [PSPM CLI](https://github.com/anyt-io/pspm-cli) for installation and usage.

## Available Skills

| Skill | Description |
|-------|-------------|
| [youtube-downloader](skills/youtube-downloader/) | Download YouTube videos and transcripts in various formats and qualities |
| [pspm-skill-creator](skills/pspm-skill-creator/) | Scaffold, validate, and package PSPM skills with isolated runtime environments |

## Repository Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Usage documentation (loaded by AI agents)
    ├── pspm.json         # PSPM manifest (name, version, metadata)
    ├── .pspmignore       # Files excluded from PSPM publishing
    └── runtime/          # Isolated execution environment
        ├── pyproject.toml / package.json
        └── <scripts>
```

Each skill is self-contained with its own isolated runtime environment — Python skills use `uv`, TypeScript skills use `bun`.

## Quick Start

### Install a skill with PSPM

```bash
pspm install youtube-downloader
```

### Run a skill manually

From the skill folder (e.g. `skills/youtube-downloader/`):

```bash
# Python skills
uv run --project runtime runtime/<script>.py [args]

# TypeScript skills
bun run runtime/<script>.ts [args]
```

## Development

See [docs/skill-development-guide.md](docs/skill-development-guide.md) for full conventions on creating new skills.

### Adding a new skill

1. Create the folder: `mkdir -p skills/<skill-name>/runtime`
2. Add `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Add `pspm.json` manifest
4. Add `.pspmignore` for dev file exclusion
5. Initialize the runtime (`uv init` for Python, `bun init` for TypeScript)
6. Add scripts, dependencies, and tests

### Validating skills

Validate a single skill:

```bash
cd skills/pspm-skill-creator
uv run --project runtime runtime/validate_skill.py ../youtube-downloader
```

Validate all skills at once:

```bash
cd skills/pspm-skill-creator
uv run --project runtime runtime/validate_all_skills.py ../
```

Validation checks:
- `SKILL.md` exists with valid YAML frontmatter
- Required fields: `name` (kebab-case, max 64 chars) and `description` (max 1024 chars, no angle brackets)
- Only allowed frontmatter keys (`name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`)
- `pspm.json` is valid JSON (if present)

### Running checks (Python)

```bash
cd skills/<skill-name>/runtime
uv run ruff check .          # Lint
uv run ruff format .         # Format
uv run python -m pyright .   # Type check
uv run pytest tests/ -v      # Test
```

### CI

A GitHub Actions workflow (`.github/workflows/validate-skills.yml`) runs automatically on pushes and PRs that touch `skills/`:

- Validates all skills via `validate_all_skills.py`
- Runs lint, type check, and tests for each skill with a Python runtime

## License

Apache-2.0
