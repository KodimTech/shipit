---
description: Deliver shipit artifacts — branch, commit, push, PR, tracker
---

Load the shipit `handoff` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/handoff/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>`.
- Read only the reference files that `SKILL.md` marks Required, from
  `<root>/skills/handoff/references/`.
- Command names are flat here: `/shipit:handoff` is `/shipit-handoff`.

This skill performs external side effects (push, PR, tracker writes). Confirm
the mode — `plan`, `implement`, or `pr-fix` — before writing anything.

Request: $ARGUMENTS
