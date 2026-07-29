# `.sdd/config.json` Schema

The machine contract. Read by `plan`, `implement`, `pr-fix`, `handoff`, `status`,
and `doctor`. Written only by `init`.

Three values carry meaning, and they are not interchangeable:

| Value | Means | Consumer behaviour |
| --- | --- | --- |
| a real value | detected and, for commands, verified | use it |
| `null` | this repo does not have it | skip that step silently |
| `"unknown"` | could not be determined | **ask the user**, never guess |

`unknown[]` at the top level lists every key that ended up `"unknown"` or was
downgraded to `null` after failing verification. `doctor` and `init` print it.

## Fields

| Key | Type | Notes |
| --- | --- | --- |
| `shipit_version` | string | Plugin version that wrote the file |
| `generated_at` | date | Used by refresh mode to detect human edits |
| `repo.name` | string | Basename of the git toplevel |
| `repo.default_branch` | string | From `origin/HEAD` |
| `repo.remote` | string | Usually `origin` |
| `docs.agent_docs` | string[] | Symlinks resolved; each file listed once |
| `docs.owned_by_shipit` | string[] | Always `[".sdd/"]` |
| `stack.languages` | string[] | With version pin files as evidence in `stack.md` |
| `stack.frameworks` | string[] | Only from declared dependencies |
| `stack.package_manager` | string \| null | Decided by lockfile |
| `commands.*` | string \| null | Never written unverified. See placeholders below |
| `commands_verified.<key>` | `{exit, at, proof}` | `proof` is the form that ran: `version`, `help`, `dry-run`, `real` |
| `paths.plans` | string | Default `.sdd/plans` |
| `paths.rules` | string | Default `.sdd/rules` |
| `paths.src` | string[] | Roots that hold product code |
| `paths.tests` | string[] | Roots that hold tests |
| `tests.framework` | string \| null | |
| `tests.location` | `mirrored` \| `colocated` \| `"unknown"` | Decided by counting files |
| `tests.naming` | string \| null | Real pattern, e.g. `*_spec.rb` |
| `tests.kinds` | string[] | Only kinds with ≥1 real file |
| `tests.coverage_gates_merge` | boolean | Whether coverage can fail the merge |
| `layers[]` | object[] | `{key, dirs[], test_kind, rule, exemplar}`. May be empty |
| `tracker.adapter` | `linear` \| `github-issues` \| `jira` \| `none` | Ambiguous → `none` |
| `tracker.issue_pattern` | string \| null | Regex, for slug and branch parsing |
| `tracker.branch_from_tracker` | boolean | Adapter can supply the branch name |
| `graph` | object \| null | `{tool, out, query, path, explain, update}`. Null unless CLI **and** graph exist |
| `markers.debt` | string | Default `ponytail:` |
| `companions.*` | `present` \| `absent` | `ponytail`, `graphify_cli`, `graphify_graph`, `caveman` |
| `worktree.enabled` | boolean | **Default `false`.** Opt-in: `plan` works in the current checkout unless the user turns this on |
| `worktree.root` | string | Default `../worktrees/<repo.name>` |
| `worktree.base` | string | Branch new worktrees fork from |
| `worktree.link[]` | string[] | **Empty at init.** Opt-in only. See warning below |
| `worktree.share_graph` | boolean | Symlink the main checkout's graph into worktrees |
| `worktree.setup` | string \| null | Runs once after a worktree is created |
| `ci.config` | string \| null | The CI file |
| `ci.required_jobs` | string[] | Jobs that gate the merge |
| `unknown[]` | string[] | Every undetermined key |

## Placeholders

Substituted at call time. Never bake the value in.

| Token | Meaning | Valid in |
| --- | --- | --- |
| `{path}` | A test file path | `commands.test_one` |
| `{base}` | `repo.default_branch` | `commands.coverage_gate`, any diff command |
| `{q}` | A natural-language question | `graph.query` |
| `{node}` | One file, symbol, or concept name | `graph.explain` |
| `{a}`, `{b}` | Two node names, in order | `graph.path` |

A `test_one` without `{path}` is malformed: `implement` cannot target a single
test, and the whole red-green loop degrades to running the full suite.

## Warning on `worktree.link[]`

Every entry becomes a symlink from a worktree to an untracked file in the main
checkout — in practice `.env` files and signing keys. Those worktrees live outside
the repo, so no `.gitignore` in the repo covers them, and any tool that runs in
the worktree root can read them.

`init` always writes `[]`. Filling it is a deliberate user decision. The safe
alternative is `worktree.setup`, which regenerates what the worktree needs.

## Reading it safely

- Never assume a key exists. A config written by an older `shipit_version` may
  lack fields; treat absent as `null`.
- Never write to this file outside `init`. A skill that wants to change the
  contract reports the drift and lets the user re-run `init`.
- A command that fails *because it does not exist* means the contract has drifted.
  Say that explicitly rather than reporting it as a code failure.
