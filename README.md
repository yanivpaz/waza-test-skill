# Waza Test Skill

This repository is a compact demo package for Waza workflows. It contains one
skill, `joker`, plus a small eval suite and several GitHub Actions workflows
that demonstrate different Waza command groups.

The skill lives at `joker/SKILL.md`. It tells concise software development
jokes when asked for programming, debugging, Git, testing, DevOps, code review,
or software engineering humor.

## Repository Layout

| Path | Purpose |
|---|---|
| `joker/SKILL.md` | The `joker` skill definition. |
| `evals/joker/eval.yaml` | Waza eval suite for the skill. |
| `evals/joker/tasks/*.yaml` | Positive and negative trigger test cases. |
| `.waza.yaml` | Waza token budget configuration. |
| `.github/workflows/*.yml` | Waza workflow demonstrations. |

## Workflows

### Waza Evaluation

File: `.github/workflows/waza-eval.yml`

Trigger:
- Push to `main` or `feature/**`
- Pull request to `main`
- Manual `workflow_dispatch`

Purpose:
- Runs the standard skill readiness and eval path that most skill repositories
  should have.
- Installs Waza with retries.
- Runs `waza check joker`.
- Runs `waza run evals/joker/eval.yaml`.
- Writes a native GitHub Step Summary with success rate, aggregate score,
  weighted score, total tests, passed/failed/error counts, and per-task scores.
- Uploads `results.json` and `results.xml`.

Artifacts:
- `waza-results-<run_id>`

### Waza Quality Gates

File: `.github/workflows/waza-quality-gates.yml`

Trigger:
- Push to `main` or `feature/**`
- Pull request to `main`
- Manual `workflow_dispatch`

Purpose:
- Demonstrates Waza validation commands that are useful as CI quality gates.
- Runs `waza check joker` in text and JSON formats.
- Runs token commands:
  - `waza tokens count`
  - `waza tokens check --strict`
  - `waza tokens profile`
- Generates an eval coverage grid with `waza coverage`.
- Generates shell completions for Bash, Zsh, Fish, and PowerShell.

Artifacts:
- `waza-quality-gates-<run_id>`
- Includes readiness output, token JSON files, coverage markdown, and completion
  scripts.

### Waza Eval Modes

File: `.github/workflows/waza-eval-modes.yml`

Trigger:
- Manual `workflow_dispatch`

Inputs:
- `trials`: number of trials for the cached parallel run.
- `workers`: number of workers for parallel mode.

Purpose:
- Demonstrates advanced `waza run` modes and result-processing commands.
- Clears the demo cache with `waza cache clear`.
- Runs a standard eval with:
  - JSON output
  - JUnit reporter
  - session logs
  - transcript output
- Runs a cached parallel eval with configurable workers and trials.
- Demonstrates task selection with `--tags` and `--task`.
- Demonstrates execution-only runs with `--skip-graders`, then grades later
  with `waza grade`.
- Demonstrates `--baseline`, `--no-skills`, `--output-dir`, and `--interpret`.
- Compares multiple result files with `waza compare`.
- Lists and views session logs with `waza session list` and `waza session view`.
- Demonstrates `waza run --discover` using a temporary colocated eval layout.
- Demonstrates `waza results list` as a storage-backed lookup. This step is
  allowed to fail because storage is intentionally not configured in this demo.

Artifacts:
- `waza-eval-modes-<run_id>`
- Includes result JSON files, JUnit XML files, session logs, transcripts,
  compare output, structured output directories, and discover-mode results.

### Waza LLM Tools

File: `.github/workflows/waza-llm-tools.yml`

Trigger:
- Manual `workflow_dispatch`

Inputs:
- `judge_model`: model used by Waza LLM-assisted commands.

Purpose:
- Demonstrates Waza commands that may require Copilot or LLM authentication.
- Runs `waza models --json`.
- Runs `waza quality joker/SKILL.md`.
- Runs `waza suggest joker --dry-run`.
- Runs `waza new task from-prompt` to generate a task file from a prompt.

Notes:
- Steps use `continue-on-error` so the workflow also documents missing-auth
  behavior instead of failing the entire demo.
- This workflow is manual because LLM-assisted commands may depend on local or
  organization authentication setup.

Artifacts:
- `waza-llm-tools-<run_id>`
- Includes model output, quality output, suggested eval YAML, and generated task
  files when authentication is available.

### Waza Serve and Scaffold

File: `.github/workflows/waza-serve-scaffold.yml`

Trigger:
- Manual `workflow_dispatch`

Purpose:
- Demonstrates non-eval Waza commands.
- Smoke-tests the HTTP dashboard server:
  - `waza serve --http --no-browser --port 3000`
- Smoke-tests the TCP JSON-RPC server:
  - `waza serve --tcp 127.0.0.1:9000`
- Demonstrates project scaffolding with `waza init`.
- Demonstrates skill scaffolding with `waza new skill`.
- Demonstrates eval scaffolding with `waza new eval`.
- Generates shell completions for Bash, Zsh, Fish, and PowerShell.

Notes:
- Server smoke tests use `timeout` and treat exit code `124` as success, because
  the server is expected to keep running until the timeout stops it.
- All scaffolded files are created in temporary directories and uploaded as
  artifacts; they are not committed to the repository.

Artifacts:
- `waza-serve-scaffold-<run_id>`
- Includes HTTP/TCP server logs, scaffold file listings, and completion scripts.

## Local Commands

Run the basic checks locally:

```bash
waza check joker --no-update-check
waza run evals/joker/eval.yaml --output results.json --reporter junit:results.xml --no-update-check
```

Run token and coverage checks:

```bash
waza tokens check joker --strict --no-update-check
waza coverage . --path joker --path evals --format markdown --no-update-check
```

Run an advanced eval locally:

```bash
mkdir -p results sessions transcripts
waza run evals/joker/eval.yaml \
  --output results/standard.json \
  --reporter junit:results/standard.xml \
  --session-log \
  --session-dir sessions \
  --transcript-dir transcripts \
  --no-update-check
```

## Installer Pattern

All workflows install Waza from the upstream installer and retry the installer
up to three times. This avoids failing demonstrations because of a transient
GitHub release lookup or download error.
