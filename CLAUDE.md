# CLAUDE.md

This repository is a **template** for dropping a consistent Claude Code setup into any new coding project — a set of review/test/docs agents, a `/work` skill that drives tasks through plan → implement → test → review → simplify, and a `docs/projects/` scaffold for specs and task lists.

When this template is copied into a real project, **replace this file** with a project-specific `CLAUDE.md`. The agents and the `/work` skill read `CLAUDE.md` to pick up project conventions, so the more specific the real `CLAUDE.md` is, the better the agents behave.

## Using This Template

1. Copy the contents of this repo into the new project root (preserving `.claude/` and `docs/`).
2. Replace this `CLAUDE.md` with one that documents the real project — see "What to put in the project's CLAUDE.md" below.
3. Replace the `README.md` with the project's own.
4. Add real projects under `docs/projects/NN-name/` using `docs/projects/templates/` as the starting point.
5. Delete this file's "Using This Template" and "What to put in the project's CLAUDE.md" sections once the project has its own content.

## What to put in the project's CLAUDE.md

The agents and `/work` skill expect the real `CLAUDE.md` to cover:

- **Project overview** — one paragraph on what the project does.
- **Architecture** — major subsystems, how they fit together, entry points.
- **Project structure** — directory tree of the main source folder with short comments. `docs-sync` updates this automatically.
- **Language / stack** — language, runtime, build system, test framework.
- **Key conventions** — import style, config access, logging, centralized wrappers (the project's equivalents of "all HTTP calls go through `src/http.ts`"). The `code-reviewer` and `architecture-reviewer` enforce these.
- **Running** — how to run, test, and build. `docs-sync` keeps this synced with the project manifest.
- **Environment variables** — list with short descriptions.
- **Agents** — table of agents in `.claude/agents/` and what they do. `docs-sync` keeps this synced.
- **Skills** — table of skills in `.claude/skills/` and what they do.
- **Threat model / security notes** — public vs private repo, sensitive assets, trust boundaries. The `security-auditor` reads this.

## What's in this template

### Agents (`.claude/agents/`)

| Agent | Purpose |
|---|---|
| `architecture-reviewer` | System-level review: module boundaries, resource lifecycle, centralization. |
| `code-reviewer` | Bug, type-safety, security, and convention review on the current diff. |
| `code-simplifier` | Flags dead code, over-abstraction, duplication, and unnecessary complexity. |
| `docs-sync` | Keeps `CLAUDE.md` and project docs aligned with the current source tree. |
| `release-notes` | Generates a changelog from recent git history. |
| `security-auditor` | Audits the diff for secrets, PII, injection risks, and unsafe I/O. |
| `test-specialist` | Writes and runs tests; bootstraps test infrastructure if missing. |

All reviewers are read-only. `docs-sync` and `test-specialist` are the only agents that modify files, and each has a bounded scope (docs-only and tests-only respectively).

### Skills (`.claude/skills/`)

| Skill | Purpose |
|---|---|
| `review` | Runs all five quality agents (`test-specialist`, `security-auditor`, `code-reviewer`, `code-simplifier`, `architecture-reviewer`) in parallel against the current uncommitted working tree and returns one consolidated verdict. |
| `work` | Drives the next unchecked task in `docs/projects/[project]/tasks.md` through the full plan → implement → test → review → simplify cycle. Supports `--auto` for unattended sweeps. |

### Project scaffold (`docs/projects/`)

- `index.md` — registry of projects with status and one-line descriptions.
- `ideas.md` — scratch list of not-yet-specced ideas.
- `templates/spec.md`, `templates/tasks.md`, `templates/test-plan.md` — starting points for a new project folder.

The `/work` skill expects each project at `docs/projects/NN-name/` with at least `spec.md` and `tasks.md`.
