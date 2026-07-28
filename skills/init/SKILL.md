---
name: init
description: Use to bootstrap or refresh the `.sdd/` contract for a repo — detects stack, verified commands, layers, test patterns, tracker, and companions from evidence. Run once per repo before `plan` or `implement`. Do not use to plan or write product code.
metadata:
  writes_product_code: false
  output: sdd-contract
---

# shipit init

Read the repo, write the contract the other skills depend on. Every other shipit
skill is repo-agnostic *because* this one is not.

Produces, at the repo root:

| Path | For | Content |
| --- | --- | --- |
| `.sdd/config.json` | machines | Commands, paths, layers, tracker, worktree, companions |
| `.sdd/stack.md` | agents | Stack facts, one row per fact, each with evidence |
| `.sdd/conventions.md` | agents | Patterns this repo actually follows, each with an exemplar |
| `.sdd/rules/<layer>.md` | agents | Per-layer rules derived from real files |
| `.sdd/rules/tests/<kind>.md` | agents | Test shape per kind, extracted from a real test |

`.sdd/` is committed. It is a team contract, not a local cache.

## Hard rules

- **No evidence, no claim.** Every line in `.sdd/*.md` carries a `path:line` or
  the exact command output that proved it. No exemplar → write `unknown`. `plan`
  then asks instead of guessing.
- **A command is written only after it ran here and exited 0** (or its
  `--version`/dry-run equivalent did). One that fails is written `null` and its
  key is appended to `unknown[]`. Never write a command you inferred from a
  README.
- **Never import conventions from another repo, another framework, or your own
  priors.** This repo's files are the only source.
- **Existing agent docs are referenced, never duplicated.** `CLAUDE.md`,
  `AGENTS.md`, `.cursor/rules/*`, `.github/copilot-instructions.md`: list them in
  `docs.agent_docs` and record in `.sdd/` only what they do *not* state. Two
  sources of truth is the failure mode this avoids.
- **Agent docs are documentation, not ground truth.** When a doc and the repo
  disagree, the repo wins and the discrepancy goes in the report.
- **Install nothing.** Detect and report. `--with-companions` and `--with-graph`
  are the only paths to a mutation, and each asks first.
- **Never write a secret** into `.sdd/`. `worktree.link[]` stays empty unless the
  user names paths explicitly.
- No product code. No branch, commit, PR, or tracker write.

## Workflow

1. **Preflight.** `git rev-parse --show-toplevel` for the root; `git rev-parse --abbrev-ref origin/HEAD` for the default branch. Not a git repo → say so and stop: tier 0 is unmet.
2. **Refresh check.** `.sdd/` already exists → switch to Refresh mode below.
3. **Detect.** Work through `references/detection-recipes.md` in order. It owns every recipe; do not improvise a detection this file does not describe.
4. **Verify commands.** Run each candidate command. Green → write it. Red or absent → `null` plus `unknown[]`. Record the result in `commands_verified`.
5. **Derive layers.** From real directories, with one exemplar each. No fixed list of layer names.
6. **Derive test rules.** One real test per kind, reduced to its shape.
7. **Detect tracker and companions.** Adapter and tier status. No installs.
8. **Write.** `config.json` from `assets/config-template.json`; the markdown files from their templates in `assets/`.
9. **Self-check.** Every `.md` line has evidence. Every non-null command has a `commands_verified` entry. `unknown[]` matches what was actually left undetermined. Fix before reporting.

## Refresh mode

`.sdd/` exists. Detection re-runs identically, then:

- Show a diff per file before writing anything.
- A file with no human edits since generation (compare against its `generated_at`
  header) is rewritten silently.
- A file that has been edited by hand is **never** overwritten without showing the
  diff and asking. Human edits outrank detection — someone knew something the
  repo does not show.
- Report what changed, what was kept, and what became `unknown` since last run.

## Flags

- `--with-companions` — print the exact install commands, state what each one
  grants (ponytail ships session hooks; `uv tool install` downloads from PyPI),
  ask once, then run `scripts/companions.sh --yes`. Never builds a graph.
- `--with-graph` — the above, plus build the graph once. Disclose the cost
  (minutes + extraction tokens) and ask separately. This is the only shipit
  action that spends tokens on setup.

## Report

- Paths written.
- Stack detected, one line.
- Commands verified, with the exit code each returned.
- **`unknown[]`, listed loudly.** Silence here is what poisons every later plan.
- Layers found, with their exemplars.
- Tracker adapter, and why.
- Companion tiers, in `doctor`'s format.
- Any place an agent doc contradicted the repo.
- Next step: `/shipit:plan`.
