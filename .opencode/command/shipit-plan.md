---
description: Plan a ticket, issue, bug or feature into an implementable contract
---

Load the shipit `plan` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/plan/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>`.
- Read only the reference files that `SKILL.md` marks Required, from
  `<root>/skills/plan/references/`. Do not read the other skills.
- Command names are flat here: `/shipit:plan` in the docs is `/shipit-plan`.
  Same for `init`, `implement`, `handoff`, `pr-fix`, `status`, `doctor`.

Request: $ARGUMENTS
