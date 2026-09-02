# Grader: the epic splits into subtasks that can each ship alone

The request spans four layers and three separately valuable outcomes. Split badly it
produces a chain of tickets where only the last one is worth merging, and the
backlog now hides that fact behind four green checkmarks.

## Pass requires all of

1. **Shape is `epic`**, decided from the request rather than asked about — it spans
   more than one entry of `layers[]` and more than one shippable outcome.
2. **Between two and seven subtasks**, each one table row, each with its own type
   and its own acceptance criterion that is observable without reading the diff.
3. **Each subtask could merge on its own** — none is "add the model", "write the
   tests", or "refactor first" as a standalone row.
4. **Dependencies are stated per row** using the local numbers, and there is no
   cycle. A subtask with no dependency is expected, not a defect.
5. **The parent carries the problem, the outcome, and the out-of-scope**, and does
   not repeat the children's acceptance criteria.
6. **The draft was written to `.sdd/tasks/<slug>.md`.**
7. **No file paths, no commands, no code** anywhere in the draft.
8. **The parent states its type** — `Feature` here — on the line under the title,
   and each subtask row carries its own.
9. **`QA steps` present**, five at most, in plain language: this change has a
   settings page, an email, and a download, all of which a person can see.
10. **The parent stays within budget** — 22 lines before the `Subtasks` table.
8. **Nothing was created at all.** The transcript ends at the draft; no tracker
   call appears in it.

## Fail on any of

- A single flat task, with the four layers listed as acceptance criteria.
- More than seven subtasks, or subtasks of subtasks.
- A "write the tests" or "set up the migration" subtask standing on its own.
- Acceptance criteria that restate the title, or that need the diff to evaluate.
- A `Files` table, a directory list, or an implementation sketch in the draft — that
  is `plan`'s output, and putting it here fossilises a guess into the backlog.
- The type missing, or left to be inferred from the prose.
- `QA steps` written with terminal commands, file paths, or framework names — the
  reader may not be a developer.
- A background paragraph, a motivation essay, or `Outcome` restating `Problem`.
- A subtask expanded into prose instead of a table row.
- Any tracker call, or an invocation of `handoff`.
- The draft claiming ids that no creation call returned.

## Notes for the judge

The exact cut is a judgement call — export request, background job, email, expiring
download, and the settings UI can reasonably group several ways. Judge the
**independence test**, not the boundaries the model chose.

Asking the user how to split it is a weaker pass, not a fail, provided the draft that
results still satisfies every point above.
