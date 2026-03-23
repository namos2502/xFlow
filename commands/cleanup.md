---
description: "Remove CortexLink configuration added by /cortexlink:setup"
---

Remove the CortexLink sections that were added during setup. Run the following steps in order.

**Step 1 — Remove from ~/.claude/CLAUDE.md:**

Read `~/.claude/CLAUDE.md`. Find the `## CortexLink` section — it starts at the `## CortexLink` heading and ends before the next `##` heading (or end of file).

- If the section **exists** — remove it (including any blank lines immediately before it) and save the file.
- If it **does not exist** — skip and note it was already absent.

**Step 2 — Remove from ~/.copilot/copilot-instructions.md:**

Read `~/.copilot/copilot-instructions.md`. Find the `## CortexLink` section using the same rule above.

- If the section **exists** — remove it (including any blank lines immediately before it) and save the file.
- If it **does not exist** — skip and note it was already absent.

**Step 3 — Confirm:**

Report which files were updated and which were already clean. Remind the user to run `/plugin uninstall cortexlink` in Claude Code to fully remove the plugin.

> **Known issue:** After uninstalling, CortexLink commands may still appear until the session is restarted. `/reload-plugins` does not remove already-loaded plugins — a full session restart may be required.
