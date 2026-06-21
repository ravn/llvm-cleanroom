# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This working directory currently contains only `AGENTS.md`. There is no source tree,
no build system, no `.git`, and no `PROJECT.md`. Treat the repo as uninitialized:
do not assume build/test/lint commands exist, and do not invent paths that aren't
on disk. When the user begins populating the project, update this file with the
real commands and architecture rather than guessing.

The directory name (`llvm-project-fix`) and references inside `AGENTS.md` to a
`lit` suite, `FileCheck`, `build-and-lit` and `runtime-tests` CI jobs, and
"compiler change" wording suggest this is intended as a workspace for an LLVM-style
fix — but none of that tooling is present here yet. Verify before relying on it.

## Working agreements

`AGENTS.md` in this directory is the human's cross-project operating contract for
all AI coding agents. Read it before any non-trivial work. Highlights that bite
in this repo specifically:

- **Verification discipline.** Mark each claim *known* vs. *guessed*. Reserve
  "confirmed" / "root-caused" for checks that would have *failed* if the claim
  were false. Building is not behaving — for behavior changes, run the runtime
  oracle before declaring done.
- **Baseline before you change.** Capture the control measurement (test fail-set,
  binary size, timing) on the unmodified system *before* touching anything.
- **CI gating.** When the project exists, a test only protects against
  regressions if CI runs it. The expected tiers per `AGENTS.md` are the lit
  suite (`build-and-lit`, primary deterministic gate, pin codegen with
  FileCheck) and `runtime-tests` (runtime oracle). Every compiler change ships
  with a lit test; add a runtime fixture when correctness is only observable at
  runtime.
- **Bug reports.** Separate verified *symptom* from *suspected cause*. Never
  write a bare "caused by Y" you haven't proven — a wrong diagnosis in an issue
  misleads the fixer.
- **PRs / commits.** Never open a PR unless explicitly asked in the current
  turn. Commit/push only when asked; branch off the default branch; delete the
  branch once merged.
- **Shell safety.** Never `cd` / `find` / `ls` / `grep` outside the project
  root without explicit instruction. Never use unquoted `===` in a shell command
  (zsh silently truncates the line) — use `---` as a separator.

If `PROJECT.md` appears alongside `AGENTS.md` later, it owns project-specific
setup, constraints, build commands, and status; this file should pick up the
fullest live detail from there.
