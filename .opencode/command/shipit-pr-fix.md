---
description: Resolve PR review comments and red CI with minimal fixes
---

Load the shipit `pr-fix` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/pr-fix/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>` —
  including `<root>/references/lean-ladder.md` and
  `<root>/skills/implement/references/validation-standards.md`.
- Command names are flat here: `/shipit:pr-fix` is `/shipit-pr-fix`.

Request: $ARGUMENTS
