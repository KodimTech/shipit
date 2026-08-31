---
name: pr-fix
description: Use when an open pull request has review comments to resolve or red CI. Minimal fix per item; scope-change requests become pushback, not code. Do not use to plan, and not for git, PR, or tracker delivery — that is `handoff`.
metadata:
  input: open-pr-feedback
  output: fix-change
  writes_product_code: true
---

# shipit pr-fix

Resolve review comments and red CI on an open PR. One minimal fix per item. No
scope expansion. Delivery — commit, push, thread replies, tracker comment — belongs
to `handoff` in `review` mode.

## Context budget

- Required: `.sdd/config.json`, the PR's unresolved review threads, and its CI
  status.
- Required: the `.sdd/rules/*` files for the layers a fix actually touches. Only
  those.
- Required: `../../references/lean-ladder.md`, relative to this skill directory,
  for the pushback tag vocabulary.
- Never bulk-load rules or docs. A three-line fix does not justify reading the
  layer's whole rule set.

## Intake — read-only, allowed

`tracker.adapter` is irrelevant here; the PR host is what matters. With `gh`
available:

1. `gh pr view <n> --json number,state,headRefName,url` — the PR exists, is open,
   and matches the current branch.
2. Unresolved threads: `gh api graphql` on `pullRequest.reviewThreads` filtered to
   `isResolved: false`. Thread state not needed →
   `gh api repos/{owner}/{repo}/pulls/{n}/comments`.
3. CI: `gh pr checks <n>`, then `gh run view <run-id> --log-failed` for each failing
   job's exact output.
4. Reproduce each failing test locally with `commands.test_one` before touching
   code.

No `gh` → ask the user to paste the review comments and the failing output. Do not
guess what a reviewer said.

## The item table

This drives the whole run. Build it before any edit.

| id | type (comment\|ci) | file | what it asks | action (fix\|pushback\|question) |
| --- | --- | --- | --- | --- |

No items → report "nothing to fix" and stop.

## Hard rules

- Minimal fix per item. No broad refactor, no new scope, no "while I'm here".
- **Prose language.** Human-facing output follows `language` in `.sdd/config.json` (absent → `en`). Code, identifiers, commit subjects and branch names stay English.
- A comment requesting a **scope or design change is not implemented.** Mark it
  `pushback` with a one-line technical justification for `handoff` to post. Use the
  ladder's tags where they fit — `yagni:` for an abstraction with one
  implementation, `native:` for a dependency the platform already covers. The
  decision stays human.
- A comment you do not understand is a `question`, not a guess.
- Never weaken or delete a test to make CI green. A red coverage gate is fixed with
  coverage, never with an exclusion.
- No fake validation. "Passed" only when the command ran and exited 0.
- No more than 3 attempts on the same failure without new evidence.
- Never run `git commit`, `git push`, a PR command, a thread reply, or a tracker
  write. Only `handoff` does that.

## Workflow

1. **Preflight.** Current branch matches the PR's `headRefName`. `git status` clean
   of changes you did not make. PR open. Wrong branch → stop and ask; never
   force-switch over user work. If the PR's branch lives in a worktree, work there.
2. **Intake.** Build the item table.
3. **Per item.** Reproduce (CI items) → confirm the expected red → minimal fix →
   targeted test green → next item. One item at a time; batching fixes makes it
   impossible to say which change resolved which thread.
4. **Validate.** The full chain from
   `../implement/references/validation-standards.md`, relative to this skill
   directory — same order, same reporting, same contract-drift rule. Exact
   commands and exit codes.
5. **Report.** The item table with a resolution per row, pushback justifications
   verbatim, a `Files Changed` manifest, and validation results.
6. **Deliver.** Validation green → invoke `handoff` in `review` mode with the
   report. Its report is appended verbatim. Do not invoke it when the run ended
   `# Fix Blocked` or `# Fix Incomplete`, or when a validation command failed.
7. **Graph sync.** `graph` set, product code changed, and you are in the **main
   checkout** → run `<graph.update>`. In a worktree, skip and say so.

## User change protection

Uncommitted changes are user-owned unless this run made them. Stop before editing
when a required file has changes you did not make, when existing changes make a fix
ambiguous, or when a fix would revert user work. Report the conflicting paths and
the decision needed. Unrelated user changes: leave them and proceed.

## Blocker conditions

- A comment is ambiguous and needs a product decision → list the exact question.
- CI is red for an external cause — flaky infrastructure, missing secrets, a runner
  problem. Report it. Never "fix" it with blind retries.
- User changes conflict with a required fix.
- The same check failed 3 times without new evidence.

`# Fix Blocked` when no product code changed yet, `# Fix Incomplete` when it did:

```markdown
# Fix <Blocked | Incomplete>

## Problem
<Concrete issue.>

## Item(s) affected
<Rows from the item table.>

## Recommended next step
<Smallest correction, or the decision needed.>

## Files inspected
- <path>
```

Include the exact failing output when the stop came from repeated failures.

## Final output

In this order: Item Table with a resolution per row · Summary · Files Changed ·
Tests Added/Updated · Validation Commands Run (exact + exit code) · Pushback
Justifications (verbatim, one per thread, for `handoff` to post) · Handoff.
