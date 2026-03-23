---
description: "Activate autonomous multi-agent mode — Claude will know when and how to invoke copilot -p or claude -p for the rest of this session"
---

⛔ **BEFORE ANYTHING ELSE — verify setup is complete:**

Read `~/.claude/CLAUDE.md` and `~/.copilot/copilot-instructions.md`. Check whether each contains a `## CortexLink` section.

If **neither** file contains a `## CortexLink` section:
> **STOP. Do not load skills. Do not activate. Do not proceed.**
> Tell the user: ⚠️ CortexLink is not set up yet. Run `/cortexlink:setup` first to detect your CLI agents and register CortexLink, then re-run `/cortexlink:auto`.

If **only one** file has the section — warn the user that setup may be incomplete and suggest re-running `/cortexlink:setup`. Then proceed.

If **both** files have the section — proceed.

---

**Load skills and activate:**

Read and load the following skill files into your context for this session:

- `skills/orchestration/SKILL.md` — the control center orchestration protocol
- `skills/agents/copilot-cli/SKILL.md` — Copilot CLI behavioral reference
- `skills/agents/claude-cli/SKILL.md` — Claude CLI behavioral reference

From now on, operate as the CortexLink control center for this session. Apply the orchestration protocol automatically whenever a task would benefit from cross-CLI delegation — without the user needing to ask explicitly.

Confirm to the user that CortexLink multi-agent mode is active. Let them know:
- You will delegate subtasks to the right CLI agent based on the decision tree
- Every delegated agent will self-verify before reporting back
- You will review each report before proceeding to the next task
- They can run `/cortexlink:setup` if any agent needs authentication
