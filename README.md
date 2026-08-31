# shipit 🐿️

Spec-driven development for any repo. It detects your conventions instead of assuming
them.

Most planning agents carry someone else's stack in their prompt. shipit carries none:
`/shipit:init` reads your repository once and writes a contract at `.sdd/`, and every
other skill works from that. Nothing in this plugin knows what framework you use.

## Install

### Claude Code

```
/plugin marketplace add KodimTech/shipit
/plugin install shipit@shipit
```

### Codex

```
codex plugin marketplace add KodimTech/shipit
codex plugin add shipit@shipit
```

Start a new session and the seven skills are available. Codex matches them by
description, and the same names work as slash commands — `/shipit:plan`,
`/shipit:init`, `/shipit:implement`, `/shipit:handoff`, `/shipit:pr-fix`,
`/shipit:status`, `/shipit:doctor` — exactly as in Claude Code. Same `skills/`
directory and the same `.sdd/` contract, so a repo initialized in one runtime
works in the others.

Update with `codex plugin marketplace upgrade shipit`; uninstall with
`codex plugin remove shipit@shipit`.

### OpenCode

The skills are plain markdown, so OpenCode runs them as custom commands:

```sh
ROOT=~/.config/opencode/plugins/shipit
[ -e "$ROOT" ] || git clone https://github.com/KodimTech/shipit "$ROOT"
"$ROOT"/scripts/install-opencode.sh
```

The installer links the seven commands and verifies OpenCode resolves them.
Re-run those three lines any time to update — the guard skips the clone, and the
installer pulls. It never uses sudo, never installs a package, and never edits
`opencode.json`.

Already have a checkout somewhere else? Skip the clone and run its
`scripts/install-opencode.sh` directly — the installer uses whatever checkout it
lives in.

Commands are flat here: `/shipit-plan`, not `/shipit:plan`. Same skills, same
`.sdd/` contract, so a repo initialized in one runtime works in the others.

Installing somewhere other than the default? Set `SHIPIT_ROOT` before running,
and keep it exported — the commands read it to find the skills. Uninstall:
`rm ~/.config/opencode/commands/shipit-*.md`.

### Then, once per repository

```
/shipit:init
```

## The cycle

```
/shipit:init        read the repo, write .sdd/          once per repo
        │
/shipit:plan        plan contract in the current checkout once per task
        │
/shipit:implement   red/green + validation scoped to the change
        │
/shipit:handoff     branch, commit, PR, tracker          the only skill with side effects
        │
/shipit:pr-fix      review comments + red CI             as needed
```

Plus two for visibility:

```
/shipit:doctor      do I have everything, and what does each gap cost
/shipit:status      every in-flight worktree, its PR, its tracker state
```

`plan`, `implement`, and `pr-fix` never run `git commit`, `git push`, a PR command,
or a tracker write. Only `handoff` does. This is enforced in every skill, so a plan
can never half-deliver itself.

## The `.sdd/` contract

`init` writes five kinds of file, and every claim in them carries evidence — a
`path:line`, or the exact command output that proved it.

| File | For | Content |
| --- | --- | --- |
| `.sdd/config.json` | machines | Verified commands, paths, layers, tracker, worktree |
| `.sdd/stack.md` | agents | Stack facts, one row per fact, each with evidence |
| `.sdd/conventions.md` | agents | Patterns this repo follows, each with an exemplar |
| `.sdd/rules/<layer>.md` | agents | Layer rules derived from real files |
| `.sdd/rules/tests/<kind>.md` | agents | Test shape, extracted from a real test |

`init` asks once, per repo, how it should be tracked:

| Choice | Means |
| --- | --- |
| Commit (default) | A team contract — visible to CI, teammates, and any other agent |
| Gitignore | Local only — each collaborator runs `init` for their own; worktrees get it via a symlink back to the main checkout |

Three rules make it trustworthy:

- **No evidence, no claim.** Without an exemplar, the value is `unknown` — and `plan`
  asks instead of guessing.
- **Commands come from CI, and are run before being written.** A command that only
  appears in a README is a candidate to verify, never a source. One that fails
  verification is `null`, and its key is reported in `unknown[]`.
- **It complements your agent docs, never duplicates them.** `CLAUDE.md`, `AGENTS.md`,
  `.cursor/rules` are referenced; `.sdd/` records only what they do not state. When a
  doc and the repo disagree, the repo wins and the discrepancy is reported.

`init` is the single point of failure by design: a bad detection poisons every later
plan. That is why the evidence rules are hard rules and why `unknown[]` is printed
loudly rather than buried in JSON.

### Output language

`init` asks once which language shipit should write in, and records it as `language`
in `.sdd/config.json` (`en` by default). It governs prose a human reads — plan,
implementation report, QA guide, PR body, tracker comments. Code, identifiers,
commit subjects, branch names and `.sdd/` itself stay English, so the repo stays
greppable for everyone. Change it by editing the key, or `/shipit:init --language es`.

### The pointer that makes it read

No agent auto-loads a directory — not `.sdd/`, not any other name. `AGENTS.md` and
`CLAUDE.md` are the files that do get loaded, so `init` writes a short pointer into
them, between `<!-- shipit:contract -->` markers:

```md
<!-- shipit:contract -->
## Repo contract (shipit)
- `.sdd/stack.md` — stack facts, each with evidence
- ...
<!-- /shipit:contract -->
```

`AGENTS.md` missing is created; present is appended once. `CLAUDE.md` missing becomes
a symlink to `AGENTS.md`, so the two never drift. Refresh replaces the block in place
— everything outside the markers is yours and is never touched.

That is what makes the contract portable across runtimes. Claude Code, Codex,
OpenCode, Cursor and anything else reading `AGENTS.md` find `.sdd/` the same way,
without shipit installed.

## Parallel worktrees — opt-in

**Off by default.** `plan` writes into your current checkout, like every other tool.
A fresh worktree does not inherit untracked local config — `.env`, credentials,
build caches — so enabling it before you know what your repo needs to boot from a
clean checkout buys friction, not parallelism.

Turn it on in `.sdd/config.json` when you actually want several tasks in flight:

```json
"worktree": { "enabled": true, "setup": "<the command that makes a clean checkout runnable>" }
```

Per-invocation override, no config edit needed — a flag in the request wins either
way: `/shipit:plan --worktree <request>` forces one, `--no-worktree` forces none.

Then `/shipit:plan` creates a worktree per task at `../worktrees/<repo>/<slug>` and
writes the plan inside it. Three things make that safe:

- **Collision detection.** Before finalizing its file list, `plan` reads the `Files`
  table of every active worktree's plan and intersects paths. Overlap is reported with
  the worktree that claims it, and the decision — sequence, narrow, or proceed — is
  recorded in the plan.
- **Shared graph.** Graph output is usually excluded via `.git/info/exclude`, so a fresh worktree has none.
  It is symlinked read-only from the main checkout. Graph updates happen only in the
  main checkout, after the merge.
- **No secrets by default.** `worktree.link[]` starts empty. Filling it symlinks
  untracked files — usually `.env` files and keys — into a directory outside the repo
  that no `.gitignore` covers. Opt in deliberately, or let `worktree.setup` regenerate
  what the worktree needs.

`/shipit:status` shows the board. `--prune` removes worktrees whose PR is merged,
never one that is dirty or still open.

## Dependency tiers

Run `/shipit:doctor` for your machine's actual state.

| Tier | What | Without it |
| --- | --- | --- |
| **0 — required** | `git` | shipit does not run. That is the whole tier. |
| **1 — recommended** | `gh` authenticated; a tracker MCP if you use one | `handoff` cannot reach GitHub. Planning and implementing are unaffected. |
| **2 — accelerators** | graphify CLI + graph; caveman | Discovery falls back to `rg`. Same answers, more tokens. |
| **3 — do not enable during a cycle** | the ponytail plugin | Nothing. The useful part is already vendored here. |

Nothing is installed for you. `doctor --fix` offers the two safe installs after one
confirmation, and never builds a graph — that costs tokens, so it stays your call.

## On ponytail

shipit vendors the useful part of [ponytail](https://github.com/DietrichGebert/ponytail)
(MIT) into `references/lean-ladder.md`: the laziness ladder, the five review tags, and
the debt-marker convention. `plan` runs the ladder as a scope gate, `implement` uses it
as a coding stance, `pr-fix` uses its tags to justify pushback.

**Do not run the ponytail plugin during an SDD cycle.** Three reasons, from its own
documentation:

1. It is a persistent mode kept alive by session hooks. shipit skills are one-shot
   workflows with their own rules — two authorities, one decision.
2. "YAGNI applies to tests too" — one `assert`, no frameworks. In a repo with a real
   suite and a coverage gate, that leaves CI red.
3. "At most three short lines" of output, against a report that must carry
   non-developer QA steps, a file manifest, and validation results.

`lean-ladder.md` overrides both of those rules explicitly. Install the plugin
separately if you want `/ponytail-audit` or `/ponytail-debt`; the debt marker prefix
stays `ponytail:` by default so its ledger still finds shipit's markers.

## Token cost

Reducing token spend is an explicit goal, not a side effect. Four levers, largest
first:

| Lever | Attacks |
| --- | --- |
| `.sdd/config.json` | Re-detecting the stack on every run. Paid once by `init`. |
| Context budget | Wide reads. Rule files load only for layers actually touched. |
| graphify | Grep dumps and whole-file reads, replaced by a scoped subgraph. |
| caveman | Prose. Non-developer QA steps stay exempt from compression; everything else, PR bodies included, is short by default. |

Plans are capped by **uncertainty**, not task size: ≤40 lines when the change follows
an analogue, ≤150 as a hard cap for novel architecture. Over budget means the plan is
restating the repo, so it gets cut rather than the cap raised.

### Measured footprint

From `claude plugin details`, against the four hand-written skills shipit was
extracted from:

| Skill | always-on | on-invoke | vs. original |
| --- | --- | --- | --- |
| `plan` | ~110 | ~1.8k | −28% on invoke (was ~2.5k) |
| `implement` | ~90 | ~2.6k | −16% on invoke (was ~3.1k) |
| `pr-fix` | ~90 | ~1.9k | +6% |
| `handoff` | ~90 | ~2k | +5% |
| `init` | ~90 | ~1.7k | new |
| `doctor` | ~90 | ~1.4k | new |
| `status` | ~70 | ~1.2k | new |

The four equivalent skills cost **~380 always-on against the originals' ~442**, and
the two heaviest got materially cheaper per invocation — the contract removed the
stack re-discovery that used to live in their bodies.

Total always-on is **~629 vs ~442**, because there are three more skills. That is the
honest trade: +187 tokens per session buys `init`, `doctor`, and `status`. If it is
not worth it for your setup, disable the plugin per-project rather than working around
it.

Neither number captures the actual saving, which is runtime input: file reads avoided
by the context budget, stack detection paid once instead of per plan, and grep dumps
replaced by a scoped subgraph. Measure that with
`claude plugin eval shipit`, which runs a no-plugin baseline arm for comparison.

## Migrating from the original skills

| Was | Now |
| --- | --- |
| `sdd-planner` | `/shipit:plan` |
| `sdd-implementation` | `/shipit:implement` |
| `sdd-pr-fix` | `/shipit:pr-fix` |
| `sdd-handoff` | `/shipit:handoff` |
| — | `/shipit:init`, `/shipit:doctor`, `/shipit:status` |
| Conventions hardcoded in the skill | `.sdd/`, generated from your repo |
| `.claude/plans/` | `<paths.plans>`, default `.sdd/plans/` |

## License

MIT. See `LICENSE`, and `NOTICE` for the ponytail attribution.
