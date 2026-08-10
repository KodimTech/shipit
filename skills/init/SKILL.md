---
name: init
description: Use to bootstrap or refresh the `.sdd/` contract for a repo — detects stack, verified commands, layers, test patterns, tracker, and companions from evidence, then points `AGENTS.md` at it so any agent loads it. Run once per repo before `plan` or `implement`. Do not use to plan or write product code.
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

## Discovery pointer

No agent auto-loads a directory. `AGENTS.md` and `CLAUDE.md` are the files that do
get loaded, so `init` writes a pointer into them. Without it `.sdd/` is a contract
nobody reads unless a shipit skill opens it by path — which is exactly the failure
mode where an agent rewrites the repo in its own conventions.

At the repo root, after the contract is written:

| State | Action |
| --- | --- |
| `AGENTS.md` absent | Create it holding the pointer block, nothing else |
| `AGENTS.md` present | Append the pointer block once, below existing content |
| `CLAUDE.md` absent | `ln -s AGENTS.md CLAUDE.md` |
| `CLAUDE.md` present, not a symlink to `AGENTS.md` | Append the pointer block once |

The block, verbatim, markers included:

````md
<!-- shipit:contract -->
## Repo contract (shipit)

Detected from this repository, not assumed. Read before writing code here.

- `.sdd/stack.md` — stack facts, each with evidence
- `.sdd/conventions.md` — patterns this repo follows, each with an exemplar
- `.sdd/rules/` — per-layer and per-test-kind rules
- `.sdd/config.json` — verified commands, layers, tracker

`unknown[]` in `config.json` lists what detection could not prove. Ask rather than
guess for anything named there.
<!-- /shipit:contract -->
````

A file already containing `<!-- shipit:contract -->` has that block **replaced in
place**, never a second copy appended. If `ln -s` fails — Windows without developer
mode — write `CLAUDE.md` as a real file holding the same block, and say which of the
two happened in the report.

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
- **The pointer never overwrites an agent doc.** Everything outside the
  `<!-- shipit:contract -->` markers belongs to the user and is left byte-identical.
  The pointer links to the contract; it never restates a fact from it.
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
9. **Point at it.** Write the pointer block per *Discovery pointer* above, then link or append `CLAUDE.md`.
10. **Self-check.** Every `.md` line has evidence. Every non-null command has a `commands_verified` entry. `unknown[]` matches what was actually left undetermined. The pointer exists exactly once per file. Fix before reporting.

## Refresh mode

`.sdd/` exists. Detection re-runs identically, then:

- Show a diff per file before writing anything.
- A file with no human edits since generation (compare against its `generated_at`
  header) is rewritten silently.
- A file that has been edited by hand is **never** overwritten without showing the
  diff and asking. Human edits outrank detection — someone knew something the
  repo does not show.
- The pointer block is replaced between its markers without asking — it is generated
  content. A missing block is re-added. Text outside the markers is never touched.
- Report what changed, what was kept, and what became `unknown` since last run.

## Flags

- `--with-companions` — print the exact install commands, state what each one
  grants (ponytail ships session hooks; `uv tool install` downloads from PyPI),
  ask once, then run `scripts/companions.sh --yes`. Never builds a graph.
- `--with-graph` — the above, plus build the graph once. Disclose the cost
  (minutes + extraction tokens) and ask separately. This is the only shipit
  action that spends tokens on setup.

## Report

- Paths written, including whether `AGENTS.md` was created or appended, and whether
  `CLAUDE.md` is a symlink or a copy.
- Stack detected, one line.
- Commands verified, with the exit code each returned.
- **`unknown[]`, listed loudly.** Silence here is what poisons every later plan.
- Layers found, with their exemplars.
- Tracker adapter, and why.
- Companion tiers, in `doctor`'s format.
- Any place an agent doc contradicted the repo.
- Next step: `/shipit:plan`.
