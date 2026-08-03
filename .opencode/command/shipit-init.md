---
description: Detect this repo's stack, commands, layers and tests into .sdd/
---

Load the shipit `init` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/init/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>`.
- Read only the reference files that `SKILL.md` marks Required, from
  `<root>/skills/init/references/` and `<root>/skills/init/assets/`.
- Command names are flat here: `/shipit:init` in the docs is `/shipit-init`.

Request: $ARGUMENTS
