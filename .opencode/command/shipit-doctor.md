---
description: Check this machine and repo for what shipit needs, by tier
---

Load the shipit `doctor` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/doctor/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>` —
  `--fix` runs `<root>/scripts/companions.sh --yes` after one confirmation.
- Command names are flat here: `/shipit:doctor` is `/shipit-doctor`.

Report by tier. Never install anything without `--fix` and a confirmation.

Request: $ARGUMENTS
