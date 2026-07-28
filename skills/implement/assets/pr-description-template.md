# PR Description Template

Exempt from prose compression. A reviewer reads this cold.

## Title

`<ISSUE-ID> <imperative summary under 60 chars>`

Follow the repo's commit convention from `.sdd/conventions.md` when it has one
(a `feat:`/`fix:` prefix, a scope). No convention recorded → plain imperative.

## Body

```markdown
## What

<2-4 sentences. The change, in reviewer terms. Not a file list — the diff already
shows files.>

## Why

<The problem this solves. Link the issue rather than restating it.>

## How

<Only the parts a reviewer would otherwise have to reverse-engineer: a non-obvious
decision, an ordering constraint, why the boring approach did not work. Skip
entirely when the diff is self-explanatory.>

## Validation

<Commands run, with exit codes. Copy from the report — do not re-word.>

## Risk

<What could break, and what would catch it. `Low — <reason>` is a complete answer
when it is true.>

## QA

<For UI changes: paste the non-developer QA steps verbatim. They were written for
someone who does not read code, and editing them for brevity destroys their
purpose.>

<For backend-only changes: `Backend only. Manual UI QA not applicable.` plus what
developer validation covered.>

## Out of scope

<What was deliberately left out, with the trigger for doing it later. Comes from
the plan's `Out of scope`. Prevents the "why didn't you also…" review round.>
```

## Rules

- Never claim a command passed that was not run.
- Never describe behaviour the diff does not contain.
- Draft PRs stay draft until `implement` reports green — marking ready is
  `handoff`'s move, not this template's.
- Screenshots for any visible change. A reviewer should not have to run the branch
  to see what it looks like.
