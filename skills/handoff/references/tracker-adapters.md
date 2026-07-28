# Tracker Adapters

Read only the section for `tracker.adapter` in `.sdd/config.json`. The others are
noise.

Every adapter answers the same four questions: what branch name, what comment, what
status transition, and what counts as terminal.

## Universal rules

- **Read the current status before writing a transition.** A blind "move to review"
  can drag a finished issue backwards.
- **Never move backwards.** Terminal and in-flight-past-this-point states are listed
  per adapter. When in doubt, leave the status alone and say so.
- **Never invent a branch prefix.** Either the adapter supplies the name, or
  `.sdd/conventions.md` has the repo's shape, or you ask.
- An unavailable tracker is `partial`, never a silent skip. Name the failing call.
- QA steps are copied **verbatim** into tracker comments. They were written for
  someone who does not read code.

## `linear`

Branch: fetch the issue and use its own `gitBranchName` — Linear's convention,
`<username>/<issue-id>-<slug>` lowercase. Unavailable → stop and report rather than
guessing a prefix.

Comment, `plan` mode:

```text
Plan: `<plan path>` — execute with /shipit:implement.
PR: <draft-pr-url>
QA: UI verification required for <named flows>
```

No UI work → `QA: Backend only. UI verification not applicable.`

Comment, `implementation` mode: summary, validation result, then the QA steps
verbatim.

Transitions: `Backlog` → `Todo` in plan mode. → the review state in implementation
mode. None in review mode.

Terminal / do not downgrade: `In Progress`, `Code Review`, `Ready for QA`,
`Accepted`, `Deployed`, `Canceled`, `Duplicate`, or that workspace's equivalents.

## `github-issues`

Branch: no tracker-supplied name. Use `.sdd/conventions.md`, else
`<user>/<issue-number>-<slug>`.

Link the issue from the PR body so GitHub closes it on merge — `Closes #<n>` when
the merge should close it, a bare `#<n>` reference when it should not. Getting this
wrong closes an issue that still has work left.

Comment: `gh issue comment <n> --body <...>`, same content as the Linear sections.

Transitions: GitHub issues have open/closed and optionally a Project field.

- Never close an issue from `handoff`. Merging does that, or a human does.
- A Project status field exists → advance it the same way as Linear, via
  `gh project item-edit`.
- No Project → labels only if the repo already uses them for state. Do not
  introduce a label scheme.

Terminal: `closed`.

## `jira`

> **Written from the API contract, not verified against a live instance.** Treat the
> exact field and transition names as unconfirmed: list the available transitions
> before choosing one, and report if they do not match what is described here.

Branch: Jira supplies no branch name. Use `.sdd/conventions.md`, else
`<user>/<ISSUE-KEY>-<slug>`. Keep the key uppercase — Jira's smart-commit and branch
detection is case-sensitive.

Comment: the same content, on the issue.

Transitions: Jira transitions are per-workflow and named per project. **Enumerate
first**, then pick by name, never by a hardcoded id:

1. List the available transitions for the issue.
2. Match on name (`In Progress`, `In Review`, `Done`).
3. No match → leave the status alone and report the available options.

Terminal: anything in a `Done` status category.

## `none`

No tracker. Delivery is git and the PR host only.

- Branch from `.sdd/conventions.md`, else ask.
- No comment step. Everything the tracker comment would have carried goes in the PR
  body instead — including the QA steps.
- No status step.
- Report the tracker lines as `n/a (adapter: none)`, not as `blocked`. Absence of a
  tracker is a configuration, not a failure.

This is the default when detection was ambiguous. A wrong adapter writes to the
wrong place, so `none` is the safe landing spot — and `/shipit:init` can be re-run
once the right one is known.
