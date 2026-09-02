# <the change, imperative, ≤70 chars>

**<Bug | Feature | Chore>** · Source: <user request | url> · <YYYY-MM-DD> · shape: <task | epic>

## Problem

<What is wrong or missing today, and for whom. One or two sentences. Cite
`path:line` when grounding found the current behaviour. No solution here.>

## Outcome

<What is true once this is done. One sentence.>

## Acceptance criteria

- [ ] <observable from outside the code, checkable without reading the diff>

## QA steps

<Only when a person can see the difference — UI, an email, a downloadable file.
Delete this section for work with no visible surface. Numbered, plain language,
written for someone who does not read code: no terminal commands, no framework
names, no file paths. Five steps at most.>

1. <where to go>
2. <what to do>
3. <what should happen>

## Out of scope

<Delete this section when nothing was excluded.>

- <excluded item> — add when <the trigger>

## Subtasks

<Epic mode only. Delete this whole section when `shape: task`. Each row must pass
the independence test in `references/split-policy.md`. One row per subtask, one
sentence per cell. Local numbers only — no tracker ids exist yet.>

| # | Type | Title | Acceptance | Depends on |
| --- | --- | --- | --- | --- |
| 1 | <bug\|feature\|chore> | <imperative, ≤70 chars> | <the one observable proof> | — |
| 2 | <bug\|feature\|chore> | <imperative, ≤70 chars> | <the one observable proof> | #1 |

## Labels / estimate

<Only labels the tracker already has. Delete this section when there are neither.>

<label>, <label> · <S | M | L> — <the biggest driver of uncertainty>

## Assumptions

<Only real ones, with the default already taken. Delete this section if empty.>

- <thing>: assumed <default>. Change is one-line if wrong.

## Blockers

<Only when the ticket cannot be created as written. A blocker carries a
recommendation. Delete this section when there are none — and when it is present,
the draft is not ready to go into the tracker yet.>

## Created

<!-- Written by `handoff` in task mode when `handoff.allow` lists `issue_create`;
     filled in by hand otherwise. Keep it, empty, until then — it is both the
     idempotency ledger for a re-run and where a later `plan` looks up the id.

     One line per issue, parent first:
       - #<local n or "parent"> — <ISSUE-ID> — <url>
-->
