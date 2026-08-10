---
description: Execute and verify an approved shipit plan
---

Load the shipit `implement` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/implement/SKILL.md` and follow it exactly. Paths it writes
  relative to itself resolve under `<root>` — including
  `<root>/references/lean-ladder.md` and
  `<root>/skills/implement/references/validation-standards.md`.
- Command names are flat here: `/shipit:implement` is `/shipit-implement`.

Request: $ARGUMENTS
