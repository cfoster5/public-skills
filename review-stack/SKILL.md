---
name: review-stack
description: Review a stacked set of GitHub PRs for the Mule-to-AWS migration using three independent lenses (team standards, claims-vs-code, Mule parity) plus a cross-stack pass. Produces draft reviews only; never edits, commits, pushes, or comments. Use when asked to review a migration PR stack, e.g. /review-stack PROJ-123.
---

# Review a migration PR stack

Usage: `/review-stack <JIRA-KEY> [--mule <path>] [--bruno <path>] [--out <dir>]`

Arguments: `$ARGUMENTS`

## Local configuration

Replace these placeholders before use:
- `MULE_APP_PREFIX`: the naming prefix of your Mule application repos
  (e.g. `acme-api` for repos like `acme-api-loans`).
- `STANDARDS_SKILL`: the name of your team's coding-standards skill,
  installed separately under `~/.claude/skills/`.

Defaults when a flag is omitted:
- `--mule`: look for `../<mule-app-name>` where the app name is the
  `MULE_APP_PREFIX-*` project named in PR 1's description. If not found, ask
  once, then proceed without it (lens C degrades; say so in the report).
- `--bruno`: look for a Bruno collection under `../bruno/` matching the
  app name. Same fallback.
- `--out`: `../<key-lowercase>-review/`

## Hard rules

- You are producing draft reviews for a human to post. Do NOT push,
  commit, post review comments, approve, or request changes on GitHub.
- Do NOT edit any file in a worktree under review. Never run `--fix`,
  `-u`, formatters, or anything that writes to tracked files.
- Subagents B and C must be launched with read-only tools
  (no Edit, Write, NotebookEdit). Lens A invokes the `STANDARDS_SKILL`
  skill; grant it the same read-only set.
- Each subagent gets only the inputs listed for it. Do not pass one
  lens's findings to another before the merge step.
- Accuracy over speed. If something cannot be verified, say UNVERIFIED;
  never guess.

## Setup

1. Discover the stack:
   `gh pr list --search "<KEY>" --state open --json number,title,headRefName,baseRefName,body`
   Order by base chain: the PR whose base is `main` is first; each
   subsequent PR's base is the previous PR's head. If the chain is
   broken or ambiguous, show it to me and stop.
2. For each PR, create a worktree: `git worktree add <out>/<nn> <headRefName>`.
3. The diff under review for each PR is `<baseRefName>..<headRefName>`.
   Never diff a non-first PR against `main`.
4. Record in `<out>/stack.json`: order, numbers, refs, and the
   description text of each PR.

## Per PR, bottom-up

Run lenses A, B, C as three concurrent subagents. Fresh context each.

### A. Standards
Invoke the `STANDARDS_SKILL` skill against the worktree, scoped to
files present in the diff. Return every violation with `file:line`,
the standard it breaks, and a one-line fix.

### B. Claims vs. code
Inputs: the PR description, the diff, shell access to the worktree.
1. Rewrite each claim in the description as a concrete, testable
   statement.
2. For each claim, locate the implementing code and label it
   PROVED (cite file:line), CONTRADICTED (cite and explain), or
   UNVERIFIED.
3. Run, check-only: `uv run pytest -q`, `uv run ruff check .`,
   `uv run mypy .`. Report exact output on failure.
4. Test quality: flag any test that would still pass if the function
   under test returned None, and any claimed behavior with no test.

### C. Mule parity
Inputs: the Mule flow XML and DataWeave for the source app, the Bruno
collection, the diff. Ignore style entirely.
For every Mule behavior this layer is responsible for (transformations,
null/empty semantics, defaults, error-to-status mapping, XML namespace
and element handling, retries/timeouts, logging of sensitive fields):
- MATCHED: cite Mule location and Python location.
- DIVERGES: cite both; give one concrete input that produces different
  output in each.
- MISSING: Mule behavior with no counterpart here; state whether a
  later layer plausibly owns it.
If `--mule` is unavailable, work from the Bruno collection and the
description only, and open the section with a line saying so.

### Merge (orchestrator, after A/B/C return)
Write `<out>/<nn>-review.md`:
1. Verdict: APPROVE / REQUEST CHANGES / NEEDS DISCUSSION, one sentence.
2. Findings, each: `[A|B|C] SEVERITY file:line` then a draft comment in
   a direct, specific voice, ready to paste. Severity is BLOCKING,
   SHOULD-FIX, or NIT. Anything flagged by two or more lenses goes
   first. BLOCKING before SHOULD-FIX before NIT.
3. Pre-existing / out of scope: issues in touched files this PR did
   not introduce.
4. Lens C coverage table: MATCHED / DIVERGES / MISSING counts.

## Cross-stack, after all PRs

One subagent in the top worktree, diffed against `main`:
- Layering: does any higher layer re-implement or bypass the foundation?
- Duplication across layers.
- Full suite: `uv run pytest -q`, `ruff`, `mypy` at the top of the stack.
- Mule behaviors still MISSING after all layers combined (union of the
  per-PR MISSING lists, minus anything a later layer provided).
Write `<out>/cross-stack.md`.

## Final output

Write `<out>/REVIEW.md`:
- One line per PR: number, title, verdict, count of BLOCKING findings.
- Then the contents of each `<nn>-review.md` in order.
- Then `cross-stack.md`.
Show me `<out>/REVIEW.md` and stop. Do not take further action.
