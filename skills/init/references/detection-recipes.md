# Detection Recipes

Run in order. Each recipe names what to look for and what evidence to record.
Do not improvise a detection this file does not describe — an invented convention
is worse than an `unknown`.

Evidence format everywhere: `path:line`, or the command plus its exit code.

## 1 — Root, default branch, remote

```bash
git rev-parse --show-toplevel
git rev-parse --abbrev-ref origin/HEAD 2>/dev/null   # → origin/main
git remote
```

No `origin/HEAD` → try `git symbolic-ref refs/remotes/origin/HEAD`, then fall
back to the current branch and record it as an assumption, not a fact.

## 2 — Language, runtime, package manager

Match manifests, then pin the manager by lockfile. A repo can hit several rows;
record all of them.

| Manifest | Language | Manager decided by |
| --- | --- | --- |
| `Gemfile` | Ruby | `Gemfile.lock` → bundler |
| `package.json` | JS/TS | `yarn.lock` → yarn · `pnpm-lock.yaml` → pnpm · `package-lock.json` → npm · `bun.lockb` → bun |
| `pyproject.toml` / `requirements.txt` | Python | `uv.lock` → uv · `poetry.lock` → poetry · `Pipfile.lock` → pipenv · else pip |
| `go.mod` | Go | go modules |
| `Cargo.toml` | Rust | cargo |
| `composer.json` | PHP | composer |
| `mix.exs` | Elixir | mix |
| `pom.xml` / `build.gradle{,.kts}` | Java/Kotlin | maven / gradle |
| `*.csproj` / `*.sln` | C# | dotnet |
| `deno.json{,c}` | TS | deno |

Runtime versions come from the pinning file, not from what is installed:
`.ruby-version`, `.node-version`, `.nvmrc`, `.tool-versions`, `mise.toml`,
`engines` in `package.json`, `rust-toolchain.toml`. Record the file.

Frameworks: infer only from a declared dependency, never from directory names.
`rails` in the Gemfile is evidence; an `app/` directory is not.

## 3 — Commands, by authority

Highest authority wins. Stop at the first source that yields a command for a key.

1. **CI config** — the only place the commands are proven to run:
   `.github/workflows/*.y*ml`, `.gitlab-ci.yml`, `.circleci/config.yml`,
   `azure-pipelines.yml`, `Jenkinsfile`, `.buildkite/`.
2. **Task runner** — `Makefile` targets, `Justfile`, `Rakefile`, `Taskfile.yml`,
   `scripts` in `package.json`, `[tool.poe]`/`[project.scripts]`.
3. **Binstubs / scripts dir** — `bin/*`, `script/*`. Executable and non-empty only.
4. **Nothing else.** A command that appears only in a README or an agent doc is a
   *candidate to verify*, never a source. If verification fails it is `null`.

Keys to fill: `setup`, `test_all`, `test_one`, `lint`, `lint_fix`, `typecheck`,
`security`, `coverage_gate`, `build`.

`test_one` must carry the `{path}` placeholder. `coverage_gate` may carry
`{base}`. Substitution happens at call time; never bake a branch name in.

Also record `ci.config` (the file) and `ci.required_jobs` (job names that gate
the merge), so `pr-fix` knows which failures matter.

## 4 — Command verification (mandatory)

For each candidate, in this order:

1. Does the executable resolve? For a binstub, is it present and executable?
2. Run the cheapest proving form: `--version`, `--help`, or a dry run.
3. For `test_one`, run it against one real existing test file.
4. Exit 0 → write the command and a `commands_verified` entry. Anything else →
   `null` + append the key to `unknown[]`.

Do not run `setup`, `build`, or a full `test_all` just to verify — resolving the
executable is enough for those. Record which proof was used.

> This recipe exists because of a real failure: a repo's agent doc documented a
> binstub that did not exist in the repo, and every plan that cited it was wrong.

## 5 — Tests

- **Framework**: from the dependency plus the CI invocation. rspec, minitest,
  jest, vitest, pytest, unittest, go test, cargo test, phpunit, exunit, junit,
  xunit.
- **Location**: mirrored (`spec/`, `test/`, `tests/` shadowing the source tree) or
  colocated (`*.test.ts` beside its source). Decide by counting real files, not
  by convention lore.
- **Naming**: the actual pattern — `*_spec.rb`, `*_test.go`, `test_*.py`,
  `*.test.tsx`.
- **Kinds**: the real subdirectories or tags that partition the suite (`model`,
  `service`, `controller`, `system`, `component`, `integration`, `e2e`, `unit`).
  Only kinds with at least one real file.
- **Coverage gate**: a tool that can fail CI on coverage —
  simplecov+undercover, `--coverage` thresholds, codecov with a required status.
  Record the command, and whether it gates the merge.

## 6 — Layers

No fixed vocabulary. Derive them:

1. List directories under each `paths.src` root, depth 1–2.
2. Keep a directory with **≥2 files** and a name that reads as a role
   (`models`, `services`, `controllers`, `components`, `jobs`, `handlers`,
   `repositories`, `usecases`, `views`, `hooks`, `stores`, `middleware`, …).
3. For each kept directory pick **one exemplar**: the file closest to the median
   size, so it shows the shape rather than the extreme.
4. Map the layer to a test kind from recipe 5 when one matches.
5. No directory qualifies → `layers: []`. `plan` falls back to per-file
   discovery. That is a valid outcome, not a failure.

Then write `.sdd/rules/<layer>.md` from `assets/layer-rule-template.md`, stating
only what the exemplar demonstrates.

## 7 — Tracker adapter

| Evidence | Adapter |
| --- | --- |
| Linear MCP tools available in session, or `[A-Z]{2,5}-\d+` in recent branch names | `linear` |
| `.github/ISSUE_TEMPLATE/`, `gh` present, issues referenced as `#\d+` in commits | `github-issues` |
| Jira MCP, or `[A-Z]+-\d+` with a Jira URL in the repo | `jira` |
| None of the above | `none` |

Record `issue_pattern` and whether the adapter can supply a branch name
(`branch_from_tracker`). Ambiguous between two → `none`, and say why in the
report. A wrong adapter makes `handoff` write to the wrong place.

## 8 — Graph

Two independent checks, both required for `graph` to be non-null:

```bash
command -v graphify
test -f graphify-out/graph.json
```

CLI + graph → fill `graph` with `{ "tool": "graphify", "out": "graphify-out",
"query": "graphify query \"{q}\"", "update": "graphify update ." }`.
CLI only → `graph: null`, and note in the report that `graphify .` would build one
(state the cost; do not run it).
Neither → `graph: null`, and no skill mentions a graph again.

Also check whether the graph output is gitignored (`git check-ignore -q
graphify-out`). If it is, set `worktree.share_graph: true` — a fresh worktree will
have no graph of its own.

## 9 — Branch and commit conventions

```bash
git log --format=%s -50
git for-each-ref --format='%(refname:short)' refs/remotes/origin --count=30
```

Conventional Commits (`feat:`, `fix:`, `chore(scope):`) in a clear majority →
record it. Branch prefixes (`feature/`, `fix/`, `<user>/<issue>-<slug>`) →
record the dominant shape. A 50/50 split is not a convention: write `unknown`.

## 10 — Agent docs

Find `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/*`, `.windsurf/rules/*`,
`.github/copilot-instructions.md`, `.clinerules/*`, `.kiro/steering/*`. List them
in `docs.agent_docs`.

Resolve symlinks: two paths pointing at one file are one doc, and loading both is
loading the same file twice.

Then **subtract**. Anything those docs already state does not go into `.sdd/*.md`.
What remains is what shipit records. Where a doc contradicts the repo, the repo
wins and the contradiction is reported.

## 11 — Companions

Report-only. See `skills/doctor/references/tiers.md` for the tier definitions and
the exact commands. Fill `companions` with `present` / `absent` per entry.

## 12 — Worktree defaults

- `enabled: false`. **Always false at init.** Worktrees are opt-in: the founder
  flips it when they actually want parallel tasks, once `worktree.setup` and the
  repo's local config are known to survive a fresh checkout. Defaulting to true
  buys friction on every first plan for a feature most repos never need.
- `root: "../worktrees/<repo-name>"`, repo name from the toplevel basename.
- `base`: the default branch from recipe 1.
- `link: []` — **always empty at init.** Only the user adds paths here, knowing
  it symlinks untracked files (often secrets) into a directory no `.gitignore`
  covers.
- `share_graph`: from recipe 8.
- `setup`: `commands.setup` if it verified green, else `null`.
