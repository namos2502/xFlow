---
description: "One-time setup — detects installed CLI agents, authenticates, and registers xFlow as always-on in ~/.claude/CLAUDE.md and ~/.copilot/copilot-instructions.md"
---

Run the following steps in order and report the results clearly to the user.

**Step 1 — Check which CLI agents are installed:**

Check for Copilot CLI:
```bash
which copilot && copilot --version
```

Check for Claude CLI:
```bash
which claude && claude --version
```

If **neither** is found, stop and show install instructions:

```bash
# GitHub Copilot CLI
brew install copilot-cli   # macOS

# Claude CLI
npm install -g @anthropic-ai/claude-code
```

Then ask the user to re-run `/xflow:setup` after installing at least one agent.

**Step 2 — Verify authentication for each installed agent:**

For Copilot CLI (if installed):
```bash
copilot -p "ping" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=claude-haiku-4.5 2>/dev/null
```
If this fails, tell the user to run `copilot login`.

For Claude CLI (if installed):
```bash
claude -p "ping" --output-format text --allowedTools "Read" --max-turns 1 --no-session-persistence 2>/dev/null
```
If this fails, tell the user to run `claude auth login`.

**Step 3 — Register xFlow in ~/.claude/CLAUDE.md:**

Read `~/.claude/CLAUDE.md` (create it if it does not exist). Check whether it already contains a `## xFlow` section.

- If the section **already exists** — skip the write and tell the user xFlow is already configured.
- If the section **does not exist** — append the following block exactly:

```markdown

## xFlow

xFlow is installed and active. It gives you a persistent control center for cross-agent CLI delegation.

**How it works:** When you take on a multi-step task, decompose it into subtasks and delegate each to the right CLI agent. Each agent self-verifies its work and returns a structured report. You review the report before proceeding.

**When to delegate cross-CLI:**
- GitHub-specific tasks (PRs, repos, Actions) → Copilot CLI
- General code tasks with context isolation or different model → Claude CLI
- Default: handle natively or use host-internal subagents

**Installed agents:**
- Copilot CLI: [✅ installed / ❌ not found]
- Claude CLI: [✅ installed / ❌ not found]

**Commands:**
- `/xflow:auto` — load all skills and activate orchestration mode explicitly
- `/xflow:setup` — re-run this setup (re-auth, update agent status)
- `/xflow:help` — show skill and command reference
```

Fill in the actual agent status from Step 1 before writing.

**Step 4 — Register xFlow in ~/.copilot/copilot-instructions.md:**

Read `~/.copilot/copilot-instructions.md` (create it if it does not exist). Check whether it already contains a `## xFlow` section.

- If the section **already exists** — skip the write.
- If the section **does not exist** — append the following block exactly:

```markdown

## xFlow

xFlow is installed and active. It gives you a persistent control center for cross-agent CLI delegation.

**How it works:** When you take on a multi-step task, decompose it into subtasks and delegate each to the right CLI agent. Each agent self-verifies its work and returns a structured report. You review the report before proceeding.

**When to delegate cross-CLI:**
- GitHub-specific tasks (PRs, repos, Actions) → Copilot CLI (use natively)
- General code tasks with context isolation or different model → Claude CLI (`claude -p`)
- Default: handle natively or use host-internal subagents

**Installed agents:**
- Copilot CLI: [✅ installed / ❌ not found]
- Claude CLI: [✅ installed / ❌ not found]

**Commands:**
- `/xflow:auto` — load all skills and activate orchestration mode explicitly
- `/xflow:setup` — re-run this setup (re-auth, update agent status)
- `/xflow:help` — show skill and command reference
```

Fill in the actual agent status from Step 1 before writing.

**Step 5 — Confirm:**

- ✅ All installed agents authenticated and both instruction files updated — tell the user xFlow is ready and always-on in Claude Code and Copilot CLI.
- ✅ Instruction files already had the section — tell the user xFlow was already configured. Offer to re-run auth check if needed.

In all cases, end with:
> 🚀 Run `/xflow:auto` to activate multi-agent mode for this session.
