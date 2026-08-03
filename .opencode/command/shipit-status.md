---
description: Show every in-flight shipit task — worktree, plan, branch, PR
---

Load the shipit `status` skill and run it.

- Plugin root: `$SHIPIT_ROOT` if set, else `~/.config/opencode/plugins/shipit`.
- Read `<root>/skills/status/SKILL.md` and follow it exactly.
- Wherever a shipit file writes `${CLAUDE_PLUGIN_ROOT}`, read it as `<root>`.
- Command names are flat here: `/shipit:status` is `/shipit-status`.

Read-only unless `--prune` is passed.

Request: $ARGUMENTS
