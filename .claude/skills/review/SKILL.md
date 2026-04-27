# /review

Run the full review panel — `test-specialist`, `security-auditor`, `code-reviewer`, `code-simplifier`, and `architecture-reviewer` — in parallel against the current uncommitted working tree, then return one consolidated verdict.

Use this as a pre-commit gate, after exploratory work outside `/work`, or any time you want a comprehensive review of "what's on disk right now."

## Usage

```
/review
```

No arguments. The skill always operates on the current working tree (staged + unstaged + untracked).

## Scope

"Uncommitted changes" means everything that differs from `HEAD`:

- **Tracked changes** — `git diff HEAD --name-only` (covers both staged and unstaged)
- **Untracked files** — entries from `git status --porcelain` whose status code is `??`

If both lists are empty, exit immediately with "Nothing to review — working tree matches HEAD." Do not launch any agents.

## Instructions

### 1. Collect the change set

Run these in parallel:

```bash
git diff HEAD --name-only
git status --porcelain
```

Build two lists:

- `tracked_changes` — output of `git diff HEAD --name-only`
- `untracked` — paths from `git status --porcelain` lines that begin with `??`

Combine into `changed_files = tracked_changes + untracked`. If `changed_files` is empty, print "Nothing to review — working tree matches HEAD." and stop.

Otherwise, print a one-line scope summary, e.g. `Reviewing 7 files (5 tracked, 2 untracked)…`, then proceed.

### 2. Launch all five agents in parallel

This is the core of the skill. **All five `Agent` tool calls MUST be issued in a single assistant turn** so the harness runs them concurrently. Do not invoke them sequentially across turns — that defeats the point of the skill.

Each prompt is self-contained: the agent gets the file list and is told to pull its own `git diff HEAD` for full content, and to read `CLAUDE.md` for project rules.

**`Agent` with `subagent_type: "test-specialist"`**
- description: `Test uncommitted changes`
- prompt:
  ```
  Run tests for the current uncommitted changes. The following files have
  changed: [list changed_files].

  1. Read CLAUDE.md to identify the project's test framework and conventions.
  2. If test infrastructure is not yet configured, bootstrap it idiomatically.
  3. Write tests for any uncovered behavior in the changed code.
  4. Run the full test suite. Fix failures you introduce; report failures
     pre-existing in the changed code without fixing unrelated tests.

  Return:
  - Suite result: PASS / FAIL (counts: N passing, M failing)
  - New test files created (paths only)
  - Any test failures with one-line cause each
  ```

**`Agent` with `subagent_type: "security-auditor"`**
- description: `Security audit uncommitted changes`
- prompt:
  ```
  Audit the current uncommitted changes for security issues. The following
  files have changed: [list changed_files]. Run `git diff HEAD` yourself to
  see the full content; for untracked files, read them directly. Read
  CLAUDE.md for the project's threat model.

  Check for: hardcoded secrets, PII exposure, sensitive content leaks, path
  traversal, injection risks (shell, SQL, template), unsafe subprocess
  usage, and anything unsafe to commit to a remote.

  Return findings tagged by severity: BLOCK (must fix before commit),
  WARN (should fix), INFO (consider). Each finding: file:line, one-line
  description, suggested fix. End with a single-line verdict:
  PASS / WARN / BLOCK.
  ```

**`Agent` with `subagent_type: "code-reviewer"`**
- description: `Code review uncommitted changes`
- prompt:
  ```
  Review the current uncommitted changes. The following files have changed:
  [list changed_files]. Run `git diff HEAD` yourself to see the full
  content; for untracked files, read them directly. Read CLAUDE.md for
  project conventions.

  Check for: bugs, null/undefined errors, race conditions, resource leaks,
  type-safety violations, error-handling gaps, and any project convention
  violations from CLAUDE.md.

  Return findings tagged by severity: BLOCK / WARN / INFO. Each finding:
  file:line, one-line description, suggested fix. End with a single-line
  verdict: PASS / WARN / BLOCK.
  ```

**`Agent` with `subagent_type: "code-simplifier"`**
- description: `Simplification check uncommitted changes`
- prompt:
  ```
  Check the current uncommitted changes for unnecessary complexity. The
  following files have changed: [list changed_files]. Run `git diff HEAD`
  yourself; for untracked files, read them directly. Read CLAUDE.md for
  intentional patterns and abstraction philosophy.

  Look for: dead code, over-abstraction, duplication, premature generality,
  unnecessary indirection, dependency bloat.

  Return findings tagged: QUICK_WIN (apply now), MEDIUM (judgment call),
  STRUCTURAL (defer / mention only). Each finding: file:line, one-line
  description, suggested change. End with a single-line verdict:
  PASS / WARN / BLOCK (BLOCK only for clearly broken patterns, not style).
  ```

**`Agent` with `subagent_type: "architecture-reviewer"`**
- description: `Architecture review uncommitted changes`
- prompt:
  ```
  Review the current uncommitted changes for architectural concerns. The
  following files have changed: [list changed_files]. Run `git diff HEAD`
  yourself; for untracked files, read them directly. Read CLAUDE.md for
  module boundaries and architectural rules.

  Check for: module boundary violations, resource lifecycle issues
  (timers, listeners, processes, file handles), centralization violations
  (bypassing the project's wrappers for HTTP, config, logging, etc.),
  startup/shutdown gaps, concurrency hazards, and improper error
  propagation across boundaries.

  Return findings tagged: BLOCK / WARN / INFO. Each finding: file:line,
  one-line description, suggested fix. End with a single-line verdict:
  PASS / WARN / BLOCK.
  ```

### 3. Synthesize the consolidated report

After all five agents return, produce one report. Do not dump raw agent output verbatim — group, dedupe, and rank.

Compute the overall verdict:

- **BLOCK** — any agent returned `BLOCK`, or `test-specialist` reported `FAIL`.
- **WARNINGS** — no BLOCKs, but at least one `WARN` finding from any agent.
- **PASS** — all five agents returned clean (`PASS` verdict, no findings, tests green).

Print the report in this format:

```markdown
# /review summary

**Scope:** N changed files (X tracked, Y untracked)
**Verdict:** PASS | WARNINGS | BLOCK

## Findings by severity

**BLOCK (count)**
- [agent] file:line — description

**WARN (count)**
- [agent] file:line — description

**INFO (count)**
- [agent] file:line — description

## Per-agent results

### test-specialist — <PASS / FAIL>
- Suite: <N passing / M failing>
- Tests written: <paths or "none">
- Failures: <list or "none">

### security-auditor — <PASS / WARN / BLOCK>
- <one-line summary; "Clean" if PASS>

### code-reviewer — <PASS / WARN / BLOCK>
- <one-line summary>

### code-simplifier — <PASS / WARN / BLOCK>
- <one-line summary>

### architecture-reviewer — <PASS / WARN / BLOCK>
- <one-line summary>

## Next steps
- <bulleted, ordered list of recommended actions; address BLOCKs first>
```

If `test-specialist` wrote new test files, call them out explicitly under "Per-agent results" and remind the user those new files were not themselves reviewed in this pass — they will be on the next `/review` run.

### 4. Stop

Do not commit, fix, or modify anything beyond what `test-specialist` already wrote. `/review` is a reporting skill — the user decides what to do with the verdict.
