---
description: Turn a need into a tracker ticket — one task, or an epic with subtasks
---

Load the shipit `task` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/task/SKILL.md` and follow it exactly.
- Read only the reference files that `SKILL.md` marks Required, from
  `<root>/skills/task/references/`. Do not read the other skills, except the one
  cross-skill reference `SKILL.md` names by path.
- Command names are flat here: `/shipit:task` in the docs is `/shipit-task`.
  Same for `init`, `plan`, `implement`, `handoff`, `pr-fix`, `status`, `doctor`.

Request: $ARGUMENTS
