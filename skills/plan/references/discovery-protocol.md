# Discovery Protocol

Goal: find analogues fast. Do not read the world. This is where a careless plan
spends ten times the tokens it needs.

## 0 — Graph first, when there is one

`graph` non-null in `config.json` **and** the output directory actually present
(inside a worktree it may be a symlink — check, do not assume):

- `<graph.query>` with the domain question → a scoped subgraph.
- `<graph.explain>` for a single file or concept.
- `<graph.path>` between two names for how they relate.

Run the command strings from `config.json` verbatim, substituting `{q}`, `{node}`,
`{a}`, `{b}`. Do not invent flags and do not load the graph tool's own skill or
docs — these three commands are the whole interface `plan` needs. A key absent
from `graph` means that command is unavailable, not that you should guess it.

Use the result to shortlist candidates, then confirm each with a real read in
step 4. **Never cite a graph node as an analogue without opening the file** — the
graph says two things are related, not that one is the shape to copy.

Nothing useful, or no graph → step 1, unchanged. Do not bulk-read a graph report;
that is for architecture review, not per-layer analogue search.

## 1 — Classify

Which layers does the request touch? Use the `layers[]` keys from
`config.json`. `layers` empty → classify by file instead, and expect to read more.

## 2 — Search by exact domain words

`rg --files` and `rg` for the request's own nouns. Real names beat guessed ones:
search the term the user used before searching the term you would have chosen.

Never read a whole directory.

## 3 — One analogue per new file

One good analogue is enough. Two or three per layer is the ceiling. Stop there.

The best analogue is the one nearest in role, not the one nearest in name.

## 4 — Read the slice, not the file

Read the relevant section. A 400-line file where 30 lines carry the pattern costs
370 lines of nothing.

## 5 — Two rounds, then decide

Two search rounds per layer maximum. After that, decide with what you have, or
record `No analogue found` plus the searches that came up empty. An honest dead
end beats a fourth round.

## 6 — Sensitive areas

Touching authorization, tenancy/account scoping, or data deletion: confirm the
relationship exists before planning around it. With a graph, its `path` command
between the model and the scoping entity. Without one, read the two files. Then
read the matching `.sdd/rules/<layer>.md` for that area only.

Never plan a scoping rule from memory of how other projects do it.

## 7 — God nodes

A file with unusually many inbound references — whether the graph flags it or `rg`
shows it — is a blast-radius warning. Slow down, widen the acceptance criteria
review, and say so in `Notes`. It is not a reason to abandon the change.

## Layer recipes

Pick the one that matches. Do not run them all. With a graph, its query goes first
in every case.

| Layer shape | Search for |
| --- | --- |
| Route / controller / handler | The route table, the URL fragment, a sibling action name |
| Service / use case / command | The layer directory plus the entry-point name from its rule file |
| Model / entity / schema | Table or collection name, the scoping association, validation and state blocks |
| View / template / component | The shared wrapper or layout name, the CSS class prefix, the slot names |
| Client-side behaviour | The controller/hook identifier, its registration point, its data attributes |
| Job / worker / consumer | The queue name, the enqueue call site |
| Migration / schema change | The most recent migration, for the shape and the safety constraints |
| Test | Exactly one existing test of the same kind. Never enumerate the test tree |

## Token discipline

- Read `.sdd/rules/*` only for layers actually touched.
- Do not open a rule file to confirm something the exemplar already showed.
- Prefer one targeted `rg` with a narrow path scope over a repo-wide sweep.
- Generated files, lockfiles, and vendored directories are never analogues.
