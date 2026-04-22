# agent-coding-setup

A drop-in template for a consistent Claude Code setup across coding projects.

Copy this repo's contents into a new project root and you get:

- **Seven agents** (`.claude/agents/`) for code review, architecture review, security audit, test writing, simplification, docs sync, and release notes.
- **A `/work` skill** (`.claude/skills/work/`) that drives the next unchecked task in a project's task list through a full plan → implement → test → review → simplify cycle, with an `--auto` mode for unattended sweeps.
- **A project scaffold** (`docs/projects/`) with templates for specs, task lists, and test plans, so the `/work` skill has a stable place to read from.

## How to use

1. Copy the contents of this repo into the new project root, preserving `.claude/` and `docs/`.
2. Replace `CLAUDE.md` with one that describes the real project — stack, conventions, architecture, env vars. The agents read it to pick up project-specific rules.
3. Replace this `README.md` with the project's own.
4. Add a first project under `docs/projects/NN-name/` using `docs/projects/templates/` as a starting point.
5. Run `/work` to pick up the first task.

See `CLAUDE.md` in this repo for the full inventory of what's included and what to put in the project's own `CLAUDE.md`.

## Layout

```
.claude/
  agents/          # review, test, docs, and release agents
  skills/
    work/          # plan → implement → test → review → simplify loop
docs/
  projects/
    index.md       # registry of projects
    ideas.md       # scratch list of pre-spec ideas
    templates/     # spec / tasks / test-plan starters
CLAUDE.md          # template for the project's CLAUDE.md
README.md          # this file — replace in real projects
```
