# Output Contract

The reader of a ticket is deciding whether to pick it up, not how to build it. Write
for that decision. `plan` will do the rest, with the repo open.

## The anti-duplication rule

**If `.sdd/*` or an agent doc already says it, the ticket does not repeat it.** The
ticket body travels into the tracker, where nobody can check it against the repo —
so a restated convention there is a copy that silently goes stale.

Never write into a ticket:

- File paths to create or edit. Owned by `plan`'s `Files` table.
- Verification commands, test structure, red/green sequencing. Owned by `implement`.
- Stack restatements, layer conventions, framework names as instructions.
- The request pasted back at the person who wrote it.
- Effort broken into hours, or a checklist of engineering steps.

## Size budget

Short by default. A ticket nobody finishes reading is a ticket nobody follows.

| Case | Target |
| --- | --- |
| Single task | **≤ 18 lines**, whole file |
| Epic parent | ≤ 22 lines, before the `Subtasks` table |
| Each subtask | **one table row.** One sentence per cell |
| `Problem` | 1–2 sentences |
| `Outcome` | 1 sentence |
| Acceptance criteria | ≤ 5, one line each |
| `QA steps` | ≤ 5 numbered steps |

Over budget means the ticket is doing `plan`'s job, or restating itself. Cut; do not
raise the cap. When `Problem` and `Outcome` would say the same thing twice, the
second one is the one to delete — not to pad into a difference.

## Required sections

The type line, `Problem`, `Outcome`, `Acceptance criteria`.

**The type is required and goes first**, on the line under the title: `Bug`,
`Feature`, or `Chore`. It is the first thing a triager filters on, and a ticket
whose type has to be inferred from the prose gets sorted wrong. Pick it from the
`Problem`: something that used to work or is visibly broken is a `Bug`; something
that does not exist yet is a `Feature`; anything with no user-visible outcome —
dependency bumps, tooling, cleanup — is a `Chore`. In epic mode each subtask carries
its own type; they need not all match the parent's.

`Subtasks` is required in epic mode and forbidden otherwise.

`QA steps` is required whenever a person can see the change — UI, an email, a
generated file — and forbidden otherwise. It is not the same artifact as the QA
guide `implement` produces: this one says how to check the ticket is done, written
before any code exists, and `implement`'s replaces it with what the change actually
did. Five steps at most, plain language, no commands and no file paths — the reader
may not be a developer.

Everything else (`Out of scope`, `Labels / estimate`, `Assumptions`, `Blockers`) is
**conditional: include only when it carries content**. Delete the heading otherwise.
Never emit `N/A` filler.

`Created` stays in the file, empty, in every mode. No skill writes it: the user
pastes the ids and urls in once the issues exist, so a later `plan` can find them
from the draft.

## Quality bar

- **The title is the change, in the imperative, ≤70 characters.** It shows up in a
  list of forty. `Expire idle sessions after 30 minutes`, not `Session bug`.
- **`Problem` says what is wrong today and for whom**, with a `path:line` when
  grounding found one. No solution in it.
- **`Outcome` says what is true when this is done.** One sentence.
- **Acceptance criteria are observable from outside the code**, and each one is
  independently checkable. If a criterion needs the diff to evaluate, rewrite it.
- **`Out of scope` names the trigger** that would bring the item back, not just the
  exclusion.
- **No sentence exists to sound thorough.** Cut every clause that would not change
  what someone does. Context the reader already has from the title is not context.
- **`Labels / estimate`** uses only labels the tracker already has — from
  `tracker.create.default_labels` or what the adapter reports. Never introduce a
  label scheme. The estimate sizes uncertainty: `S` / `M` / `L` plus the driver.

## Prohibited output

- Product code, pseudocode, schemas, endpoint signatures.
- A file manifest, or "touch these directories".
- "Decide later" placeholders for core behaviour.
- Acceptance criteria that restate the title.
- A restated `Problem` under `Outcome`, or a `QA step` that repeats an acceptance
  criterion word for word.
- Background paragraphs, motivation essays, or a summary of the conversation the
  need came from.
- Ceremony: self-validation checklists, risk tables with no concrete risk,
  definition-of-done boilerplate the tracker already enforces.

## Blocking vs assuming

Same two categories as `plan`, and no third:
`../../plan/references/ambiguity-policy.md`. A blocker stops the draft before
delivery — a ticket that ships with a blocking unknown just moves the question to
whoever picks it up.

## Self-check before finalizing

Run these as checks. Do not emit them into the draft.

- Required sections present; conditional sections carry content or are gone.
- Within size budget — count the lines, do not estimate them.
- Type stated, and it matches what `Problem` describes.
- `QA steps` present if and only if the change is visible to a person.
- Title ≤70 characters, imperative, specific.
- Every acceptance criterion is checkable without reading the diff.
- No file paths, no commands, no code.
- Epic: every subtask passes the independence test in `split-policy.md`.
- `Created` present and empty.
