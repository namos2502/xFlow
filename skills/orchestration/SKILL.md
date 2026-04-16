---
name: orchestration
description: This skill should be used when a multi-step task can benefit from cross-CLI delegation — decomposing work into subtasks, routing them to Copilot CLI or Claude CLI agents, and synthesizing their structured reports back to the user.
user-invocable: false
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
ONE AGENT PER SUBTASK — never delegate the same subtask to more than one agent.
Claude Code → Copilot CLI (GitHub tasks) or Claude CLI (last resort for isolated tasks).
Copilot CLI → Claude CLI.
```

## The Two-Tier Model

```
CONTROL CENTER (your AI agent — decomposes, routes, reviews)
  ├── AGENT (Copilot CLI)
  ├── AGENT (Claude CLI)
  └── AGENT (future-cli ...)
        └── [own tools & sub-agents — internal, platform-native]
```

The control center IS your active AI agent — it holds the plan and directs the work. Each Agent is a full CLI agent in its own right, not a dumb executor. The tree is one level deep: Agents are peers, they don't chain to each other. Width scales as you add agents; depth stays fixed.

## Task Complexity

Classify every task before routing. This determines how much spec detail to write and whether a Q&A phase is needed.

| Level | Signals | Delegation style |
|-------|---------|-----------------|
| **Simple** | Single operation, read-only, unambiguous output | Handle inline — no cross-CLI |
| **Standard** | 2–4 operations, may write, clear success criteria, low rework cost | Full spec: problem + acceptance criteria |
| **Complex** | 5+ operations, writes/commits/PRs, judgment calls, OR high rework cost | Full spec + Q&A turn before execution |

**Key signal for Complex:** rework cost. Tasks with irreversible steps (PRs, commits, deploys) or required judgment calls always qualify.

## When to Delegate Cross-CLI

Each subtask goes to exactly **ONE agent** — the decision tree picks which one, then stop. Never route the same subtask to both agents simultaneously.

**Peer direction by host:**
- **Claude Code** → Copilot CLI for GitHub tasks; Claude CLI only as last resort (when context isolation or a specific Anthropic model is needed AND native tools are insufficient)
- **Copilot CLI** → Claude CLI for code tasks, analysis, and refactors

**Delegate when:**
1. **Platform-specific** — GitHub ops (PRs, repos, Actions) → Copilot; Anthropic reasoning or Claude-specific model → Claude CLI
2. **Context isolation** — offload a long subtask so its intermediate work never enters your context
3. **Different model needed** — the target CLI runs a model the host cannot
4. Task is **Standard or Complex** — the work inside the agent justifies the delegation overhead

**Do NOT delegate when:**
1. Task needs your current session context, open files, or in-memory state
2. The host has a native subagent that can handle it
3. The target CLI is not installed or not authenticated
4. You have no clear reason — "big task" is not a reason
5. Task is **Simple** — handle inline; a full agent session costs more than the task itself

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
2. **Route** — Apply decision tree. Check agent availability (see `references/report-format.md`).
3. **Dispatch** — Use delegation prompt template (see `references/delegation-template.md`). Always include scope, success criteria, report format. Each subtask goes to ONE agent — never dispatch the same work to two agents. Always run the target CLI in programmatic mode (`copilot -p` / `claude -p`); never open an interactive terminal session.
4. **Review** — Read STATUS first. Spot-check if needed (`git diff`, tests). Decide: proceed, re-assign, or adjust.
5. **Track** — Update state (done / pending / failed). Never skip to the next subtask without reviewing the current report.
6. **Synthesize** — Consolidate into one output for the user. Lead with issues (🔴 blocker / 🟠 should fix / 🟡 minor), then a one-sentence verdict. If any subtask is ❌, hold the verdict until resolved.

## Red Flags — STOP

- Delegating without a clear reason ("it's a big task")
- Moving to the next subtask without reading the report
- Returning raw output instead of a structured report
- Marking ✅ without running verification
- Retrying a ❌ task without reading ISSUES first
- Delegating a task that needs current session context
- Agents chaining to each other (all coordination goes through the control center)
- Delegating the same subtask to more than one agent (one subtask → one agent, always)
- Opening an interactive terminal session for delegation (always run the target CLI with `-p` programmatic mode)
- Claude Code delegating to Claude CLI when native tools (Task tool, inline work) would suffice

## Quick Reference

| I want to… | Do this |
|------------|---------|
| GitHub task | Copilot — load `cortexlink:copilot-cli` via the Skill tool |
| General code task | Claude CLI — load `cortexlink:claude-cli` via the Skill tool |
| Delegation prompt | See `references/delegation-template.md` |
| Report format / self-verify | See `references/report-format.md` |
| Add a new agent | Drop `skills/agents/<name>/SKILL.md` — follow existing format |
| Handle auth failure | Tell user: `/cortexlink:setup` |
| Handle ❌ report | Read ISSUES. Re-assign or handle natively. Never retry blindly. |

