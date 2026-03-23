---
name: orchestration
description: Cross-agent orchestration protocol. Teaches the host AI agent to act as a persistent control center — decomposing tasks, fanning out to peer CLI agents, receiving verified reports, and synthesizing results.

---

# CortexLink Orchestration

## Overview

CortexLink turns your active AI agent into a **control center** that fans out tasks to peer CLI agents. Each agent executes in isolation, self-verifies, and returns a structured report. You stay in context — results come back, intermediate noise stays out.

**Core principle:** Delegate with a clear reason. Review every report before proceeding.

**Violating the letter of this protocol is violating its spirit.**

## The Iron Law

```
NO CROSS-CLI DELEGATION WITHOUT A CLEAR REASON.
NO PROCEEDING WITHOUT REVIEWING THE REPORT.
```

## The Two-Tier Model

```
CONTROL CENTER (your AI agent — decomposes, routes, reviews)
  ├── AGENT (Copilot CLI)
  ├── AGENT (Claude CLI)
  └── AGENT (future-cli ...)
        └── [own tools & sub-agents — internal, platform-native]
```

The control center IS your active AI agent — it holds the plan and directs the work. Each Agent is a full CLI agent in its own right, not a dumb executor. It can use its own platform-native tools and sub-agents internally to complete the task. The tree is one level deep: Agents are peers, they don't chain to each other. Width scales as you add agents; depth stays fixed.

## Task Complexity

Classify every task before routing. This determines how much spec detail to write and whether a Q&A phase is needed.

| Level | Signals | Delegation style |
|-------|---------|-----------------|
| **Simple** | Single operation, read-only, unambiguous output (a list, a status, a count) | Handle inline — no cross-CLI |
| **Standard** | 2–4 operations, may write, clear success criteria, low rework cost | Full spec: problem + acceptance criteria |
| **Complex** | 5+ operations, writes/commits/PRs, judgment calls, OR high rework cost if misunderstood | Full spec + Q&A turn before execution |

**Key signal for Complex:** rework cost. If the agent misunderstands and you must redo the work — how expensive is that? Tasks with irreversible steps (PRs, commits, deploys) or required judgment calls always qualify.

## When to Delegate Cross-CLI

**Delegate when:**
1. **Platform-specific** — GitHub ops (PRs, repos, Actions) → Copilot; Anthropic reasoning or Claude-specific model → Claude CLI
2. **Context isolation** — offload a long subtask so its intermediate work never enters your context (the final result DOES return via stdout)
3. **Different model needed** — the target CLI runs a model the host cannot
4. Task is **Standard or Complex** — the work inside the agent justifies the delegation overhead

**Do NOT delegate when:**
1. Task needs your current session context, open files, or in-memory state
2. The host has a native subagent that can handle it
3. The target CLI is not installed or not authenticated
4. You have no clear reason — "big task" is not a reason
5. Task is **Simple** — handle inline or use a native subagent; a full agent session costs more than the task itself

**Decision tree:**
```
New task →
  Simple? → handle inline (no cross-CLI)
  Host native subagent available? → use it (faster, no auth needed)
  Platform-specific (GitHub, Anthropic API, etc.)? → cross-CLI
  Context isolation needed (verbose output, long subtask)? → cross-CLI
  Different model needed? → cross-CLI
  Default → handle inline

  When going cross-CLI:
    Standard → full spec (problem + acceptance criteria)
    Complex → full spec + Q&A turn before execution
```

## Control Center Protocol

1. **Decompose** — Break into scoped, independently executable subtasks. Each must be verifiable by the agent itself.
2. **Route** — Apply decision tree. Check agent availability (see below).
3. **Dispatch** — Use delegation prompt template. Always include scope, success criteria, report format.
4. **Review** — Read STATUS first. Spot-check if needed (`git diff`, tests). Decide: proceed, re-assign, or adjust.
5. **Track** — Update state (done / pending / failed). Never skip to the next subtask without reviewing the current report.
6. **Synthesize** — When all subtasks are done, consolidate into one output for the user. Never surface raw agent output. Lead with issues (🔴 blocker / 🟠 should fix / 🟡 minor), then a one-sentence verdict on what to do next. If any subtask is ❌, hold the verdict until resolved.

## Delegation Prompt Template

```
[Task]: <one sentence — what needs to be done>
[Context]: <only what the agent strictly needs>
[Success criteria]: <how the agent knows it's done>
[Report format]: Return ONLY this when finished. Plain text labels only — no **bold**, no # headers:
  STATUS: ✅ Verified / ⚠️ Partial / ❌ Failed
  SUMMARY: <1-2 sentences>
  STEPS: <bullets>
  FILES: <changed files or "none">
  ISSUES: <notes for the control center or "none">
  Max 150 words.
```

Keep prompts lean. No project dumps.

**For Complex tasks only**, append this block to the delegation prompt before the report format:

```
[Before executing]: This task is complex. In your first response, list any questions
or ambiguities — 1 turn only. Do not perform any actions until I confirm.
```

The agent surfaces questions, you refine the spec if needed, then issue the execution follow-up.

## Structured Report Format

Every agent, every tier, every time:

```
STATUS: ✅ Verified / ⚠️ Partial / ❌ Failed
SUMMARY: <1-2 sentences>
STEPS:
  - <step 1>
  - <step 2>
FILES: <list or "none">
ISSUES: <notes or "none">
```

**Never return raw output or tool call dumps to the control center.**

## Self-Verify Before Reporting (when YOU are the agent)

Execute → Verify → (fix if needed) → Report.

1. Run the task
2. Check your own work — tests, file contents, expected state
3. If verification fails → fix, then verify again
4. Only then → write the structured report

| Status | Meaning |
|--------|---------|
| ✅ Verified | You actively checked. It works. |
| ⚠️ Partial | Some parts verified, some could not be checked. Explain in ISSUES. |
| ❌ Failed | Describe what was attempted and what failed. |

**Never mark ✅ without actually checking.**

## Agent Detection

```bash
# Copilot CLI
which copilot 2>/dev/null && copilot -p "ping" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=claude-haiku-4.5 2>/dev/null | head -1

# Claude CLI
which claude 2>/dev/null && claude -p "ping" --output-format text --allowedTools "Read" --max-turns 1 --no-session-persistence 2>/dev/null | head -1
```

If unavailable: fall back to another agent, handle natively, or tell the user to run `/cortexlink:setup`.

## Error Handling

| STATUS | Control center action |
|--------|-----------------------|
| ✅ | Accept, proceed |
| ⚠️ | Read ISSUES, accept partial or re-assign unverified parts |
| ❌ | Read ISSUES. Re-assign, adjust scope, or handle natively |
| No output | Retry once. If still failing, handle natively. |
| Auth failure | Tell user: `/cortexlink:setup` |

**After 2 consecutive ❌ on the same subtask — stop and discuss with the user before retrying.**

## Cross-Agent Chaining

Fan out to multiple Agents in parallel when tasks are independent. Pass a report excerpt (SUMMARY + STEPS) as context into the next delegation prompt — never raw output. Agents do not chain to each other; all coordination happens at the control center.

**Avoid duplicate fetches:** When multiple agents need the same source data (e.g. a PR diff, a file's contents), fetch it once in the control center and pass it as `[Context]` in each delegation prompt. Do not let each agent re-fetch the same data independently.

## Red Flags — STOP

- Delegating without a clear reason ("it's a big task")
- Moving to the next subtask without reading the report
- Returning raw output instead of a structured report
- Marking ✅ without running verification
- Retrying a ❌ task without reading ISSUES first
- Delegating a task that needs current session context
- Agents chaining to each other (all coordination goes through the control center)

## Quick Reference

| I want to… | Do this |
|------------|---------|
| GitHub task | Copilot — see `skills/agents/copilot-cli/SKILL.md` |
| General code task | Claude CLI — see `skills/agents/claude-cli/SKILL.md` |
| Add a new agent | Drop `skills/agents/<name>/SKILL.md` — follow existing format |
| Handle auth failure | Tell user: `/cortexlink:setup` |
| Handle ❌ report | Read ISSUES. Re-assign or handle natively. Never retry blindly. |
