---
name: orchestration
description: This skill should be used when a multi-step task can benefit from parallel subtask execution or cross-CLI delegation — decomposing work into independent subtasks, routing them to native subagents or CLI agents in parallel, and synthesizing their structured reports back to the user.
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
INDEPENDENT SUBTASKS RUN IN PARALLEL — never serialize by default.
PREFER NATIVE SUBAGENTS — cross-CLI only when platform-specific or context isolation is needed.
Claude Code → native Agent tool first → Copilot CLI (GitHub) → Claude CLI (last resort).
Copilot CLI → Claude CLI.
```

## The Two-Tier Model

```
CONTROL CENTER (your AI agent — decomposes, routes, reviews)
  ├── NATIVE SUBAGENT (Agent tool — preferred for all general tasks)
  ├── AGENT (Copilot CLI — GitHub/platform-specific tasks only)
  ├── AGENT (Claude CLI — last resort: Anthropic-specific model or context isolation)
  └── AGENT (future-cli ...)
        └── [own tools & sub-agents — internal, platform-native]
```

The control center IS your active AI agent — it holds the plan and directs the work. Native subagents (Agent tool) are the first choice: no auth, no process overhead, full tool access. Cross-CLI agents are full CLI agents in their own right — not dumb executors — but are for platform-specific work only. The tree is one level deep: agents are peers, they don't chain to each other. Width scales as you add agents; depth stays fixed.

## Task Complexity

Classify every task before routing. This determines how much spec detail to write and whether a Q&A phase is needed.

| Level | Signals | Delegation style |
|-------|---------|-----------------|
| **Simple** | Single operation, read-only, unambiguous output | Handle inline — no cross-CLI |
| **Standard** | 2–4 operations, may write, clear success criteria, low rework cost | Full spec: problem + acceptance criteria |
| **Complex** | 5+ operations, writes/commits/PRs, judgment calls, OR high rework cost | Full spec + Q&A turn before execution |

**Key signal for Complex:** rework cost. Tasks with irreversible steps (PRs, commits, deploys) or required judgment calls always qualify.

## Concurrency First

After decomposing, map dependencies before dispatching. **Default to parallel — serialize only when a subtask needs another's output.**

- **Parallel:** no shared state, no output dependency → dispatch all in one batched message
- **Sequential:** subtask B consumes A's report → block on A first, then dispatch B

Serializing independent subtasks doubles wall-clock time for no gain. When background agents are running, do prep work for synthesis — never idle-block.

## Concurrency Primitives

| Primitive | How | Use for |
|-----------|-----|---------|
| Multiple `Agent` calls in one message | Emit N `Agent` tool calls in a single response | **Primary parallel fan-out** — native subagents |
| `Agent` + `run_in_background: true` | Set on the `Agent` tool call | Long-running native subagent; collect via `Read` on output file |
| Shell `cmd1 & cmd2 & wait` in `Bash` | Standard shell backgrounding, write to temp files | `copilot -p` / `claude -p` fan-out only |

> `run_in_background: true` is a parameter of the **`Agent` tool**, not the `Bash` tool.

**Preferred order for subtasks:**
1. Native `Agent` tool — no auth, no process overhead, preferred for all general work
2. `copilot -p` — GitHub/platform-specific tasks only
3. `claude -p` — Anthropic-specific model or context isolation only

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
2. The host has a native subagent that can handle it — **always try native Agent first**
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

1. **Decompose** — Break into scoped, independently executable subtasks. Each must be verifiable by the agent itself. After decomposing, identify which subtasks are independent — these are your parallel batch.
2. **Route** — Apply decision tree. Prefer native Agent tool. Check CLI agent availability only if cross-CLI is needed (see `references/report-format.md`).
3. **Dispatch** — Batch all independent subtasks into ONE message. For native subagents: emit multiple `Agent` tool calls in one response. For cross-CLI: background with shell `&` in a single `Bash` call, write output to temp files. Use delegation prompt template for cross-CLI (see `references/delegation-template.md`); always include scope, success criteria, and report format. Each subtask goes to ONE agent — never dispatch the same work to two agents.
4. **Useful-wait** — While background agents run, do prep work for synthesis or the next subtask. Never idle-block when there is useful work to do.
5. **Review** — Read STATUS first. Spot-check if needed (`git diff`, tests). Decide: proceed, re-assign, or adjust.
6. **Track** — Update state (done / pending / failed). Never skip to the next subtask without reviewing the current report.
7. **Synthesize** — Consolidate into one output for the user. Lead with issues (🔴 blocker / 🟠 should fix / 🟡 minor), then a one-sentence verdict. If any subtask is ❌, hold the verdict until resolved.

## Red Flags — STOP

- Delegating without a clear reason ("it's a big task")
- Moving to the next subtask without reading the report
- Returning raw output instead of a structured report
- Marking ✅ without running verification
- Retrying a ❌ task without reading ISSUES first
- Delegating a task that needs current session context
- Agents chaining to each other (all coordination goes through the control center)
- Delegating the same subtask to more than one agent (one subtask → one agent, always)
- Opening an interactive terminal session for delegation (always use the `-p` prompt flag)
- Reaching for `copilot -p` / `claude -p` when a native Agent tool call would do
- Dispatching agents one at a time when subtasks are independent (serializing for no reason)
- Idle-blocking on agent output when there is useful prep work to do

## Green Flags — Do These

- ≥2 independent subtasks → batch all into one message, dispatch simultaneously
- General code task → native Agent tool first, not Claude CLI
- Background agent running → do prep work, don't idle-block
- Native Agent can handle it → skip cross-CLI entirely

## Quick Reference

| I want to… | Do this |
|------------|---------|
| Run independent subtasks in parallel | Multiple `Agent` tool calls in one message |
| Long-running background native work | `Agent` + `run_in_background: true`; collect via `Read` |
| GitHub task | Copilot — load `cortexlink:copilot-cli` via the Skill tool |
| General code task (native first) | Native Agent tool — emit multiple `Agent` calls in one message |
| General code task (cross-CLI) | Claude CLI — load `cortexlink:claude-cli` (last resort only) |
| Delegation prompt | See `references/delegation-template.md` |
| Report format / self-verify | See `references/report-format.md` |
| Agent context protocol | See `references/agent-context.md` |
| Add a new agent | Drop `skills/agents/<name>/SKILL.md` — follow existing format |
| Handle auth failure | Tell user: `/cortexlink:setup` |
| Handle ❌ report | Read ISSUES. Re-assign or handle natively. Never retry blindly. |

