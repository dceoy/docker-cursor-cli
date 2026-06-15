# AGENTS.md

Guidelines for AI coding agents working in this repository.

## Project overview

This repository provides a Docker Compose workflow for running
[Cursor CLI](https://cursor.com/docs/cli/overview) (`cursor-agent`) — Cursor's
AI-powered coding assistant — inside an isolated, reproducible container.
The container runs as a non-root `agent` user, mounts the current repository at
`/workspace`, and persists Cursor CLI and application state in named Docker volumes.

## Repository structure

```
.
├── Dockerfile        # Multi-stage build: base → cli → cursor
├── compose.yml       # Docker Compose service definition
├── .github/
│   ├── dependabot.yml
│   ├── renovate.json
│   └── workflows/
│       └── ci.yml
├── AGENTS.md         # This file
└── README.md
```

## Development workflow

Use Docker Compose for all local workflows.

Build the image:

```bash
docker compose build
```

Start an interactive Cursor CLI session:

```bash
docker compose run --rm cursor-cli
```

Run a one-off command without entering a session:

```bash
docker compose run --rm cursor-cli -lc 'cursor-agent --version'
```

## Code standards

- **YAML** — 2-space indentation, lowercase keys.
- **Dockerfile** — uppercase instructions; all `RUN` layers use
  `bash -euo pipefail` for strict error handling.
- **Shell scripts** — use `#!/usr/bin/env bash` with `set -euo pipefail`.
- Keep each Dockerfile `RUN` layer focused on a single logical step to
  maximise layer caching.

## Testing approach

There are no unit tests. Validation is operational:

1. `docker compose build` — confirms the image builds cleanly.
2. `docker compose run --rm cursor-cli -lc 'cursor-agent --version'` — smoke-tests
   the installed CLI.
3. CI runs Hadolint (Dockerfile linter) and other static checks automatically.

Run local QA before opening a pull request.

## Security

- **Never commit API keys or tokens.** Supply credentials exclusively through
  environment variables at runtime or interactive sign-in.
- Cursor CLI authenticates via Cursor sign-in (`cursor-agent login`) or a Cursor
  API key. When `CURSOR_API_KEY` is unset, run `cursor-agent login` on first
  launch to print an authorization URL to complete in a browser on your local
  machine.
- The default Compose command runs `cursor-agent` with `--force`, which lets the
  agent make file changes and run shell commands without confirmation. This is
  intended for the disposable container only; drop the flag to restore Cursor's
  default approval-gated behaviour.
- The following secrets are consumed from the host environment and forwarded
  into the container:

  | Variable         | Purpose                                               |
  | ---------------- | ----------------------------------------------------- |
  | `CURSOR_API_KEY` | Authenticate Cursor CLI with a Cursor API key         |
  | `GITHUB_TOKEN`   | Authenticate `gh` CLI operations inside the container |

## Contribution guidelines

- Write clear, descriptive commit messages explaining _why_ a change was made.
- Reference related issues or pull requests in the PR description.
- Document the local validation steps you ran (build, smoke test, QA) in the
  PR body.
- Keep changes minimal — avoid unrelated refactors in the same PR.
