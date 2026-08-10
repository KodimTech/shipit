---
name: doctor
description: Use to check whether this machine and repo have what shipit needs — git, GitHub CLI, tracker, graph tool, prose compression — reported by tier with the exact fix command and its cost. `--fix` installs the safe ones after one confirmation.
metadata:
  writes_product_code: false
---

# shipit doctor

The single answer to "do I have everything?". Reports by tier, with the exact
command to fix each gap and what that command costs.

Tier 0 is one item: `git`. Everything else is optional, and the report says what you
lose rather than implying breakage.

## Hard rules

- **Default is report-only.** Nothing installed, nothing mutated, no file written.
- `--fix` runs `../../scripts/companions.sh --yes`, relative to this skill
  directory, after **one** confirmation that lists what will run and what each
  command grants.
- **Never build a graph.** Not on `--fix`, not on any flag. It costs tokens, and
  that is the user's call. Print the command; stop there.
- Never `sudo`. Never pipe a downloaded script into a shell. Never install a package
  manager.
- A missing optional tool is reported as `missing`, never as an error, and never with
  language implying shipit is broken.
- `ponytail` present is reported as something to **turn off**, not as a success.
- Do not write to `.sdd/config.json`. `init` owns the `companions` block; this skill
  only reads and reports.

## Workflow

1. Run `scripts/companions.sh` (no flags) and use its output as the source of truth.
   It is versioned and auditable; do not re-implement its checks inline.
2. Add the repo-side checks the script cannot make: is `.sdd/` present, does
   `config.json` parse, is `unknown[]` non-empty, does `commands.test_one` carry
   `{path}`.
3. Print the report below.
4. `--fix` → show what will run, ask once, then invoke the script with `--yes`.
   Re-run step 1 afterwards and print the new state.

## Report format

```
tier 0  git                 ok       2.51.0
tier 1  gh                  ok       authenticated as <login>
        tracker             linear   MCP connected
tier 2  graphify cli        ok       ~/.local/bin/graphify
        graphify graph      missing  → graphify .        cost: minutes + tokens
        graph in worktree   n/a      symlinked from main checkout
        caveman             missing  → prose stays uncompressed
tier 3  ponytail            absent   correct — ladder is vendored

contract
        .sdd/               ok       generated 2026-07-28
        unknown[]           2 keys   typecheck, security
        test_one            ok       carries {path}

1 optional gap. shipit works. discovery uses rg until a graph exists.
```

Rules for the last line: state what still works before what is missing. A user
reading this should know whether they can proceed, in one line.

## What each gap actually costs

| Gap | Consequence |
| --- | --- |
| `git` | shipit cannot run. Only real blocker. |
| `gh` | `handoff` cannot open or update PRs. Planning and implementing are unaffected. |
| tracker `none` | No issue comments, no status transitions. Everything else works. |
| graphify CLI | Discovery falls back to `rg`/`git grep`. Same answers, more tokens. |
| graphify graph | Same as above — having the CLI without a graph changes nothing. |
| caveman | Chat prose and reports stay long. Note that shipit exempts non-developer QA steps and PR bodies from compression either way. |
| `.sdd/` absent | `plan` and `implement` refuse to run. Fix: `/shipit:init`. |
| `unknown[]` non-empty | Those verification steps do not exist. Named in every implement report. |
| `ponytail` **present** | Its persistent mode contradicts spec-driven tests and truncates reports. Fix: `stop ponytail`. |

## Why `--fix` cannot do everything

Installing `gh` or `uv` needs a system package manager, and choosing one for you is
not this skill's call. Those are printed as commands.

The two mutations `--fix` will make: `uv tool install graphifyy` when `uv` is already
present, and a `claude plugin install` for a companion you approved. Both are stated
before they run.

See `references/tiers.md` for each tier's definition and the full command list.
