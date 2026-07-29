---
name: implement
description: Use when an approved shipit plan must be executed, verified, and prepared for QA. Runs the repo's own verified commands from `.sdd/config.json`. Do not use to redesign scope, and not for git, PR, or tracker delivery — that is `handoff`.
metadata:
  input: sdd-plan
  output: implemented-change
  writes_product_code: true
---

# shipit implement

Execute the approved plan. The plan is the contract. The code is minimal, scoped,
and verified with commands that are known to run in this repo.

## Context budget

- Required: the full plan, `.sdd/config.json`, `.sdd/conventions.md`.
- Required: the `.sdd/rules/<layer>.md` for each layer the plan touches, and the
  `.sdd/rules/tests/<kind>.md` for each test it plans. Only those.
- Required: `${CLAUDE_PLUGIN_ROOT}/references/lean-ladder.md`. Fallback:
  `../../references/lean-ladder.md` from this skill directory.
- Required: `references/validation-standards.md`. Every run validates.
- Optional, only if touched: `references/security-standards.md`.
- Never bulk-load every rule file or every agent doc. Resolve symlinked docs once.
- If a rule was skipped, do not claim compliance with it.

## Hard rules

- Require an approved plan. Missing → stop and ask.
- No scope expansion. No redesign. No new abstraction. No dependency upgrade.
- No behaviour change outside the plan.
- No weakening of authorization, scoping, security, or existing tests.
- No edits to credentials, secrets, env files, or production config unless the plan
  explicitly requires and approves it.
- No TODO standing in for required behaviour.
- **No fake validation.** "Passed" only when the command actually ran and exited 0.
- No more than 3 attempts on the same failing test, lint, or check without new
  evidence. After that, stop and report.
- Never run `git commit`, `git push`, a PR command, or a tracker write. Delivery
  happens only by invoking `handoff`.
- Non-developer QA steps and the PR body are **exempt from prose compression**.
  Write them in full even under a terse-output mode.

## Workflow

1. **Preflight.** Read the plan. If it has a `Worktree` section, confirm you are in
   the worktree it names — wrong location is the most common way this run edits the
   wrong branch. No such section → the current checkout is correct; do not create a
   worktree and do not go looking for one. Check
   `git status` and detect changes you did not make. Inspect only the files the
   plan names. Search for more only when the plan lacks detail; a graph query
   before `rg` when `graph` is set and its output is present.
2. **Contract gate.** The plan must have a goal, acceptance criteria, a `Files`
   table with analogues, tests mapped to rule files, `Manual QA` when UI is
   touched, and no unresolved blockers. Missing → stop with
   `# Implementation Blocked`. Plans are deliberately terse: verification commands,
   red/green sequencing, and repo conventions are owned by this skill and by
   `.sdd/*`, not by the plan. Their absence is correct, not a gap.
3. **Test first.** Where practical: write the failing test → run it with
   `commands.test_one` → confirm the expected red → minimal code → re-run → next
   layer. An **unexpected** red is investigated before any production change.
   `commands.test_one` null or missing `{path}` → say the loop is degraded and run
   `test_all` instead. Do not silently skip the loop.
4. **Implement.** The smallest correct change. Repo patterns over new structure.
   Apply the ladder as stance, not as licence to deviate from the plan. A deliberate
   shortcut leaves a debt marker with its ceiling and upgrade trigger.
5. **Validate.** `references/validation-standards.md`. Report exact commands and
   exit codes.
6. **Docs sync.** The plan has a `Docs impact` section → apply each listed edit,
   scoped to the named sections, and list the docs in `Files Changed`. No doc
   rewrites. If the implementation drifted in a way that hits a docs trigger the
   plan missed, update the doc and note it under `Known Risks or Follow-ups`.
7. **QA guide.** UI changed → convert the plan's `Manual QA` into plain-language
   browser steps: URL and environment, which role to log in as, setup data,
   numbered actions, expected results, screenshots needed, pass/fail notes. No
   framework names, no class names, no terminal commands. Backend only → say
   `Backend only. Manual UI QA not applicable.` and state what developer
   validation covered instead.
8. **PR text.** Compose the title and body from
   `assets/pr-description-template.md` into the report. Write it; do not post it.
9. **Deliver.** Validation green and the report written → invoke `handoff` in
   `implementation` mode and pass it the report. Its report is appended to yours
   verbatim, never summarized into a claim it did not make. Do not invoke it when
   the run ended `# Implementation Blocked` or `# Implementation Incomplete`, or
   when any validation command failed.
10. **Graph sync.** `graph` set, product code changed, and **you are in the main
    checkout** → run `<graph.update>`. Inside a worktree, skip it and say so: the
    graph is shared, and updating it from a branch writes branch-local state into
    every other worktree's view. It is updated in the main checkout after the merge.

## User change protection

Uncommitted changes are user-owned unless this run made them. Stop before editing
when a required file has changes you did not make, when existing changes make the
plan ambiguous or risky, or when the plan would revert user work. Report the
conflicting paths, why they conflict, and the exact decision needed. Unrelated user
changes: leave them alone and proceed.

## Allowed changes

Plan-named code, tests, templates, styles, client-side code, migrations, routes,
policies, services, and jobs. Doc edits listed in `Docs impact`, scoped to the
named sections. Small support changes required to make planned behaviour work.
Lint fixes in files you already touched. Tests demanded by the coverage gate for
behaviour you changed.

## Forbidden changes

Broad refactors, unrelated cleanup, dependency upgrades, UI redesigns, new
abstractions with no plan reason. Public behaviour change outside the plan. Test
rewrites outside planned behaviour. Deleting or weakening tests to go green.
Touching the user's uncommitted work.

## Blocker conditions

Stop when: the plan is missing, has unresolved blockers, or lacks contract detail;
acceptance criteria contradict each other; security or scoping is unspecified; the
plan requires destructive data work without explicit approval; the plan conflicts
with `.sdd/*`; user changes conflict with planned edits; a major decision is needed
that the plan does not contain; or the same check failed 3 times without new
evidence.

`# Implementation Blocked` when no product code changed yet,
`# Implementation Incomplete` when it did. Both use:

```markdown
# Implementation <Blocked | Incomplete>

## Problem
<Concrete issue.>

## Why the current plan is unsafe or incorrect
<Technical risk.>

## Recommended correction
<Smallest correction needed.>

## Files inspected
- <path>
```

Include the exact failing command output when the stop came from repeated failures.

## Final output

Use `assets/implementation-report-template.md`. Sections in that order: QA Handoff,
QA Steps For Non-Developer, Plan, Summary, Files Changed, Tests Added/Updated,
Validation Commands Run, Security Notes, Handoff, Known Risks or Follow-ups,
PR Preparation.
