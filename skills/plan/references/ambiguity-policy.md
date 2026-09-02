# Ambiguity Policy

Two categories, and only two. Every uncertainty is either an assumption with a
default, or a blocker. There is no "open question" middle ground — that is how a
plan ships with a hole in it.

## Blocking

Stop the plan and emit `Blockers` when getting it wrong would be **expensive or
unsafe to reverse**:

- **Authorization** — who may do this, and whether the check is real or cosmetic.
- **Scoping** — which tenant, account, or owner a record belongs to, when the
  request does not say.
- **Data shape** — a destructive migration, a column type that will hold live
  data, an irreversible backfill.
- **Money** — amounts, rounding, currency, tax, refunds.
- **External contract** — a payload another system already consumes.
- **Contradictory acceptance criteria** — two criteria that cannot both hold.
- **Silent security downgrade** — the only way to satisfy the request as written
  weakens an existing check.

Format:

```markdown
## Blockers

- <question>. Needed because <what breaks if guessed>. Options: <A> / <B>.
  Recommendation: <one of them>, because <reason>.
```

Always carry a recommendation. A blocker with no recommendation makes the user do
the analysis you already did.

## Assuming

Everything else. Record it in `Assumptions` **with the default already taken**, so
the implementer proceeds:

- Copy, labels, empty-state wording.
- Sort order, pagination size, default filter.
- Which of two equally conventional file locations to use.
- Log level, metric name.
- Whether an optional field is shown when absent.

Format:

```markdown
## Assumptions

- <thing>: assumed <default>. Change is one-line if wrong.
```

If reversing the assumption would not be one-line, it is probably a blocker.
That is the test.

## Not ambiguity

These look like ambiguity and are not. Resolve them yourself:

- The convention exists in `.sdd/*` → follow it.
- An analogue shows the pattern → copy it.
- `config.json` has the command → use it.
- The repo has a dominant pattern → follow the majority.
- `.sdd/conventions.md` lists it under **Not conventions** → *that* is real
  ambiguity. Ask, or assume with a default per the rules above.

## When `.sdd/` says `unknown`

An `unknown` in the contract is `init` telling you it could not determine
something. Do not fill the gap by guessing, and do not silently pick the
convention from another project.

- `unknown` on a **command** → the plan cannot rely on that verification step.
  Note it; `implement` will report the gap.
- `unknown` on a **test kind or layer shape** → ask, unless an analogue in the
  actual diff area settles it.
- `unknown` on the **tracker** → treat as `none`. Never guess an adapter;
  the branch and the PR would point at the wrong place.

An `unknown` that keeps blocking plans is a signal to re-run `/shipit:init`, not
to work around it forever. Say so.
