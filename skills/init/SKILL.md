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

Whether `.sdd/` is committed or stays local is a decision made once, at the first
`init` — recorded in `sdd_tracking`. Committed is the default: a team contract,
not a local cache, visible to CI, teammates, and any other agent. Local keeps
the contract private to this checkout via `.git/info/exclude`; each collaborator
runs `init` for their own.

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
- **`language` is asked, never detected.** The language a repo's docs are written
  in does not imply the language its owner wants shipit to write in. Ask once,
  default `en`, record it in `language`.
- **`sdd_tracking` is asked once.** First `init` only, before anything is written.
  Refresh reads the recorded value and never re-asks except on a discrepancy
  against real git state.
- **Local tracking uses `.git/info/exclude`, never `.gitignore`.** `.gitignore` is
  itself a shared, committed file — writing to it to keep something "local" is a
  contradiction. `.git/info/exclude` lives under `.git/`, is never committed, and
  the decision stays exactly as private as intended, in every collaborator's own
  checkout.
- No product code. No branch, commit, PR, or tracker write.

## Workflow

1. **Preflight.** `git rev-parse --show-toplevel` for the root; `git rev-parse --abbrev-ref origin/HEAD` for the default branch. Not a git repo → say so and stop: tier 0 is unmet.
2. **Refresh check.** `.sdd/` already exists → switch to Refresh mode below.
3. **Ask tracking.** First `init` only — there is no recorded decision yet. Ask
   whether `.sdd/` should be committed (default — a team contract visible to CI,
   teammates, and any other agent) or stay local (nobody else sees it; each
   collaborator runs `init` themselves; a task's plan file won't travel as a
   PR diff either — `handoff` pastes it into the PR body instead). Write the
   answer to `sdd_tracking`. Local → check `git check-ignore -q .sdd` first, and
   append a `.sdd/` entry to `.git/info/exclude` only if nothing already covers
   it — never to `.gitignore`.
4. **Ask the output language.** First `init` only, in the same breath as tracking.
   Which language should shipit write in — plan, report, QA guide, PR body,
   tracker comments? Default `en`. Accept a tag (`es`, `pt-BR`) or a plain name;
   store the tag in `language`. Prose only: code, identifiers, commit subjects,
   branch names and `.sdd/` itself stay English. See
   `references/config-schema.md § Scope of language`.
5. **Detect.** Work through `references/detection-recipes.md` in order. It owns every recipe; do not improvise a detection this file does not describe.
6. **Verify commands.** Run each candidate command. Green → write it. Red or absent → `null` plus `unknown[]`. Record the result in `commands_verified`.
7. **Derive layers.** From real directories, with one exemplar each. No fixed list of layer names.
8. **Derive test rules.** One real test per kind, reduced to its shape.
9. **Detect tracker and companions.** Adapter and tier status. No installs.
10. **Write.** `config.json` from `assets/config-template.json`; the markdown files from their templates in `assets/`.
11. **Point at it.** Write the pointer block per *Discovery pointer* above, then link or append `CLAUDE.md`.
12. **Self-check.** Every `.md` line has evidence. Every non-null command has a `commands_verified` entry. `unknown[]` matches what was actually left undetermined. The pointer exists exactly once per file. Fix before reporting.

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
- `sdd_tracking` absent — config predates this field — → ask, same as a first
  `init`, before continuing refresh.
- `sdd_tracking` present → read it, never re-asked — unless the repo state
  disagrees: recorded `local` but `git ls-files .sdd | head -1` returns a
  tracked path, or `git check-ignore -q .sdd` now fails. Either disagreement is
  reported and asked again, same as a hand-edited file.
- `language` is a human decision, not a detection: keep the recorded value, never
  re-ask. `--language` overrides it; a config predating the field gets `en`.
- Report what changed, what was kept, and what became `unknown` since last run.

## Flags

- `--language <tag>` — set `language` without asking. For CI and re-runs.
- `--with-companions` — print the exact install commands, state what each one
  grants (ponytail ships session hooks; `uv tool install` downloads from PyPI),
  ask once, then run `scripts/companions.sh --yes`. Never builds a graph.
- `--with-graph` — the above, plus build the graph once. Disclose the cost
  (minutes + extraction tokens) and ask separately. This is the only shipit
  action that spends tokens on setup.

## Report

- Paths written, including whether `AGENTS.md` was created or appended, and whether
  `CLAUDE.md` is a symlink or a copy.
- Tracking: committed or local, and whether `.git/info/exclude` was touched.
- Stack detected, one line.
- Output language recorded, and whether it came from the question or `--language`.
- Commands verified, with the exit code each returned.
- **`unknown[]`, listed loudly.** Silence here is what poisons every later plan.
- Layers found, with their exemplars.
- Tracker adapter, and why.
- Companion tiers, in `doctor`'s format.
- Any place an agent doc contradicted the repo.
- Next step: `/shipit:plan`.
