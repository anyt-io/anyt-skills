---
name: create-anyt-notebook
description: Create AnyT Notebook (.anyt.md) files for AI agent workflows. Use when the user wants to create, scaffold, or generate an AnyT notebook, design an AI workflow with cells, or build a step-by-step notebook for tasks like project setup, data pipelines, or content generation.
---

# Create AnyT Notebook

Create `.anyt.md` notebook files — workflow documents for AI agents that combine task cells (AI execution), shell cells (scripts), input cells (user forms), note cells (documentation), and break cells (review gates).

For the complete specification, see [references/spec-v2.0.md](references/spec-v2.0.md).
For product context, see [references/product.md](references/product.md).
For real-world examples, see files in [references/examples/](references/examples/).

## File Format

An `.anyt.md` file has two sections: YAML frontmatter and a body with cell tags.

```
---
[YAML frontmatter]
---

# [Notebook Title]

[Cell tags in execution order]
```

## Frontmatter

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `schema` | `"2.0"` | Yes | — | Must be first field, always `"2.0"` |
| `name` | string | Yes | — | Notebook identifier (used as heading) |
| `description` | string | No | — | Human-readable summary |
| `version` | string | No | — | Semantic version |
| `workdir` | string | No | `"anyt_workspace"` | Working directory for execution (relative to notebook file) |
| `inputs` | object | No | `{}` | Key-value pairs available to all cells as context |
| `env_file` | string | No | `".env"` | Path to .env file for secrets |
| `dependencies` | object | No | — | Skill dependencies as `name: version` pairs |

Minimal valid frontmatter:

```yaml
---
schema: "2.0"
name: my-notebook
workdir: output
---
```

### Working Directory

`workdir` sets the current working directory for all cell execution. All file paths in shell scripts and task descriptions should be **relative to workdir**, not include the workdir name.

```yaml
workdir: my-project
```

Correct: `mkdir -p src docs` (creates `my-project/src`, `my-project/docs`)
Wrong: `mkdir -p my-project/src` (creates `my-project/my-project/src`)

### Input Values

```yaml
inputs:
  projectName: My Project        # string
  maxRetries: 3                  # number
  verbose: true                  # boolean
  port:                          # schema object
    type: number
    required: true
    default: 3000
    description: Server port
    value: 3000
```

### Environment Variables

Never put secrets directly in the notebook. Use `env_file` to reference an external `.env` file (excluded from version control):

```yaml
env_file: ".env"              # Default
env_file: "secrets/prod.env"  # Relative path
```

Priority (highest to lowest): notebook inputs (prefixed `ANYT_`) > .env file > shell profile.

## Cell Types

All cells use XML-like tags with a required `id` attribute:

```xml
<{type} id="{unique-id}">
{content}
</{type}>
```

Rules:
- Valid types: `task`, `shell`, `input`, `note`, `break`
- `id` is required, must be unique across the notebook
- ID format: lowercase alphanumeric + hyphens (e.g., `setup-env`, `create-api`)
- No nesting of cell tags
- Only valid attribute is `id` — never include `status`, `duration`, `error`
- Leave a blank line between cell tags

### Task Cell

AI agent executes natural language instructions. Markdown formatting is preserved.

```xml
<task id="create-api">
Create a REST API with the following endpoints:
- GET /users - List all users
- POST /users - Create a new user

Use Express.js with TypeScript and proper error handling.

**Output:** src/app.ts, src/routes/users.ts
</task>
```

- Reference files created by previous cells (paths relative to workdir)
- Reference input cell responses and frontmatter inputs
- Use `**Output:** file1, file2` to declare expected output files

### Shell Cell

Executes bash scripts directly — no AI involvement. Fast, deterministic.

```xml
<shell id="setup-env">
mkdir -p src tests docs
npm init -y
npm install express typescript @types/express
</shell>
```

- `#!/bin/bash` shebang is optional but recommended
- Exit code 0 = success, non-zero = failure
- Use for installation, building, testing, deployment

### Input Cell

Pauses execution and displays a form. Resumes when user submits.

```xml
<input id="project-config">
## Configure Your Project

<form type="json">
{
  "fields": [
    {
      "name": "projectName",
      "type": "text",
      "label": "Project Name",
      "required": true,
      "validation": { "minLength": 3, "pattern": "^[a-z][a-z0-9-]*$" }
    },
    {
      "name": "database",
      "type": "select",
      "label": "Database",
      "default": "sqlite",
      "options": [
        { "value": "sqlite", "label": "SQLite" },
        { "value": "postgres", "label": "PostgreSQL" }
      ]
    },
    {
      "name": "features",
      "type": "multiselect",
      "label": "Features",
      "options": [
        { "value": "auth", "label": "Auth" },
        { "value": "api", "label": "API" }
      ],
      "validation": { "minItems": 1 }
    },
    {
      "name": "port",
      "type": "number",
      "label": "Dev Port",
      "default": 3000,
      "validation": { "min": 1024, "max": 65535 }
    },
    {
      "name": "isPublic",
      "type": "checkbox",
      "label": "Public Repository"
    }
  ]
}
</form>
</input>
```

**Field types:** `text`, `textarea`, `number`, `checkbox`, `select`, `radio`, `multiselect`

**Field properties:**

| Property | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Unique field identifier |
| `type` | Yes | Field type |
| `label` | Yes | Display label |
| `description` | No | Help text |
| `required` | No | Whether field must be filled |
| `default` | No | Default value |
| `placeholder` | No | Placeholder text |
| `options` | Conditional | Options array for select/radio/multiselect: `[{ "value": "v", "label": "L" }]` |
| `rows` | No | Textarea rows (default: 3) |
| `validation` | No | Rules: `minLength`, `maxLength`, `pattern`, `min`, `max`, `step`, `minItems`, `maxItems` |

**Input without form** (action buttons only):

```xml
<input id="review">
## Review Output

Check the generated files and choose how to proceed.
</input>
```

### Note Cell

Markdown documentation checkpoint. Auto-completes instantly during execution.

```xml
<note id="phase-1-complete">
## Phase 1 Complete

Generated files:
- `src/app.ts` - Express server
- `src/routes/` - API routes

Next: Configure database and add authentication.
</note>
```

### Break Cell

Pauses execution for human review. Waits for user to click "Continue."

```xml
<break id="verify-setup">
Review the project structure and configuration files
before proceeding to the database setup.
</break>
```

## Design Patterns

### Linear Pipeline
```
shell (setup) -> task (generate) -> task (process) -> note (summary)
```

### Human-in-the-Loop
```
input (config) -> task (generate) -> break (review) -> task (refine)
```

### Mixed Automation
```
shell (mkdir, npm install) -> task (generate code) -> shell (build) -> task (write tests)
```

### Phased Execution
```
note (Phase 1) -> task -> task -> break (review)
note (Phase 2) -> input -> task -> shell -> note (done)
```

## Validation Rules

When generating a notebook, ensure:

1. Frontmatter starts with `---` and ends with `---`
2. `schema: "2.0"` is the first field in frontmatter
3. `name` is required
4. `workdir` should be set to a meaningful directory name
5. Every cell `id` is unique within the file
6. Only valid cell types: `task`, `shell`, `input`, `note`, `break`
7. Every `<type id="...">` has a matching `</type>`
8. No `status`, `duration`, `error`, or `exitCode` attributes on cell tags
9. IDs are slug-format: lowercase, alphanumeric, hyphens
10. Include `# {name}` heading after frontmatter, matching `name`

## Complete Example

```yaml
---
schema: "2.0"
name: express-api
description: Build an Express.js API with TypeScript
workdir: express-api
---

# express-api

<shell id="init">
mkdir -p src tests
npm init -y
npm install express typescript @types/express
npx tsc --init
</shell>

<task id="create-server">
Create an Express.js server with TypeScript:
- src/app.ts with a basic Express setup
- src/routes/health.ts with a GET /health endpoint
- tsconfig.json configured for Node.js

**Output:** src/app.ts, src/routes/health.ts, tsconfig.json
</task>

<shell id="build">
npx tsc
</shell>

<break id="verify">
Check that the TypeScript compiles without errors and review the generated code.
</break>

<task id="add-tests">
Add Vitest tests for the health endpoint:
- Install vitest and supertest as dev dependencies
- Create tests/health.test.ts
- Add a "test" script to package.json
</task>

<shell id="test">
npm test
</shell>

<note id="complete">
## API Ready

- Server: `src/app.ts`
- Routes: `src/routes/health.ts`
- Tests: `tests/health.test.ts`

Run: `npx tsc && node dist/app.js`
</note>
```

## Workflow Tips

- **Start with a note cell** to describe the workflow overview
- **Use shell cells** for deterministic steps (install deps, build, test) — cheaper and faster than AI
- **Add break cells liberally** during development, remove once confident
- **Keep task cells focused** — one clear objective per cell
- **Declare outputs** in task cells with `**Output:** file1, file2` so the system tracks expected files
- **Reference previous cells** — task cells can reference files created by earlier cells and input responses
- **Use input cells** to collect user configuration before AI tasks that depend on choices
- **Set workdir** to keep all generated files organized in one directory
