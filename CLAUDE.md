# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this project.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:6cd5cc61 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Agent Context Profiles

The managed Beads block is task-tracking guidance, not permission to override repository, user, or orchestrator instructions.

- **Conservative (default)**: Use `bd` for task tracking. Do not run git commits, git pushes, or Dolt remote sync unless explicitly asked. At handoff, report changed files, validation, and suggested next commands.
- **Minimal**: Keep tool instruction files as pointers to `bd prime`; use the same conservative git policy unless active instructions say otherwise.
- **Team-maintainer**: Only when the repository explicitly opts in, agents may close beads, run quality gates, commit, and push as part of session close. A current "do not commit" or "do not push" instruction still wins.

## Session Completion

This protocol applies when ending a Beads implementation workflow. It is subordinate to explicit user, repository, and orchestrator instructions.

1. **File issues for remaining work** - Create beads for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **Handle git/sync by active profile**:
   ```bash
   # Conservative/minimal/default: report status and proposed commands; wait for approval.
   git status

   # Team-maintainer opt-in only, unless current instructions forbid it:
   git pull --rebase
   git push
   git status
   ```
5. **Hand off** - Summarize changes, validation, issue status, and any blocked sync/commit/push step

**Critical rules:**
- Explicit user or orchestrator instructions override this Beads block.
- Do not commit or push without clear authority from the active profile or the current user request.
- If a required sync or push is blocked, stop and report the exact command and error.
<!-- END BEADS INTEGRATION -->


## Build & Test

Requires `gmake` (FreeBSD) and `uv`; Go 1.24+ for `impl/go`.

```bash
gmake help          # list run/demo/test/lint/verify targets
gmake test          # pytest across impl/python (+ hy/clojure/guile if present)
gmake verify        # bin/verify.sh — the conformance gate (verify the verifier, then every impl vs the traces)
gmake lint          # ruff / clj-kondo / byte-compile, per language
gmake check         # lint + test + verify

# Go (impl/go): the faithful d54 port, the display model, and the TUIs
cd impl/go && go build ./... && go vet ./... && go test ./...
```

## Architecture Overview

`SPEC.md` is the one canonical, versioned spec; every other document defers to it. The game is rebuilt once per language under `impl/<lang>/` (Python is the legacy oracle, then Hy → Clojure → Elisp → Guile), each held to the shared conformance traces in `spec/conformance/`. `impl/go/` is a separate lineage: a faithful Go port of the original 2012 d54 Java (`mitris`), plus an in-process model of the `wal.sh/tools/display` protocol and two TUIs. `contrib/` holds display tooling — `displays/` (wal.sh WebSocket lineage), `d54-sim/` (original TCP-arcade lineage), `emacs/`. See [docs/POLYGLOT-PLAN.md](docs/POLYGLOT-PLAN.md) and [docs/PROCESS.md](docs/PROCESS.md).

## Conventions

- **SPEC.md wins.** A derived doc that disagrees with it is the bug. Never edit `SPEC.md`, the traces, or `spec/SEALS.md` outside the seal protocol (POLYGLOT-PLAN.md, "Seal and rebuild").
- **Docs are Markdown here** (even normative `SPEC.md`/`PROTOCOL.md`); match the surrounding files, don't add `.org`.
- **Conventional commits** (`feat(scope):`, `fix:`, `docs:`); concise, no emoji. Stage files by name — never `git add -A`.
- **Displays:** never connect to the live relay `wss://wal.sh/tools/display/ws` from here (PROTOCOL.md §5.4); test against the local mock relay.
- Commits that touch `SPEC.md`, the traces, or the gate carry a `git notes` entry (Timeline + Reproduction + supported cell).
