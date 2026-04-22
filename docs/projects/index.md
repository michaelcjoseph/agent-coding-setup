# Projects

Each project lives in a numbered folder (`NN-name/`) with a `spec.md`, a `tasks.md`, and optionally a `test-plan.md`. The `/work` skill drives tasks in these folders through a plan → implement → test → review → simplify cycle.

| Project | Status | Description |
|---|---|---|
| _No projects yet._ | | Add a folder under `docs/projects/NN-name/` using the files in `templates/` as a starting point. |

## Conventions

- Folder name: `NN-short-slug` (e.g., `01-mvp`, `02-auth-refresh`). Two-digit prefix keeps ordering stable.
- `spec.md`: what the project is, requirements, and implementation phases.
- `tasks.md`: the unit the `/work` skill consumes. Each line is a meaningful deliverable with a `- [ ]` / `- [x]` checkbox.
- `test-plan.md` (optional): priority-tagged error-handling scenarios for the test-specialist to cover.
- Status values: `Specced` → `In progress` → `Done`.
