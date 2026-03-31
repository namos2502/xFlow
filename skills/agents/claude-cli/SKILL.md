---
name: claude-cli
description: This skill should be used when delegating general code tasks — explanations, refactors, fixes, analysis — via `claude -p`. Also use when context isolation or a specific Anthropic model is needed for the subtask.
user-invocable: false
---

# Claude CLI Agent

## When to Use

⛔ If the host is **Claude Code**: use Claude CLI only when you specifically need context isolation or a different Anthropic model AND native tools (Task tool, inline work) are insufficient. Claude Code → Claude CLI is last resort. Prefer Copilot CLI for GitHub tasks; prefer Claude Code's native Task tool for general code tasks.

---

**Use Claude CLI for:**
- General code tasks — explanations, refactors, fixes, analysis
- Tasks needing Anthropic-specific models (Sonnet, Opus, Haiku)
- Context isolation — offload long subtasks without polluting the host context
- Tasks where you want a different model than the host is running

**Do NOT use Claude CLI for:**
- GitHub-specific tasks (PRs, repos, Actions) — use Copilot CLI
- Tasks that need your current session context or open files
- When Claude CLI is not installed or not authenticated

## Core Flags

| Flag | Purpose |
|------|---------|
| `-p "PROMPT"` | Print mode — non-interactive, exits when done |
| `--output-format text` | Plain text output (default — easiest to parse) |
| `--output-format json` | Structured JSON with metadata |
| `--no-session-persistence` | Do not save session to disk — keeps scripted calls clean |
| `--max-turns N` | Cap agentic turns — prevents runaway tasks |

## Tool Permissions

| Flag | Purpose |
|------|---------|
| `--allowedTools "TOOL"` | Tools that auto-execute without a permission prompt |
| `--disallowedTools "TOOL"` | Tools removed from the model's context entirely |

| Use case | Flags |
|----------|-------|
| Questions, analysis | `--allowedTools "Read"` |
| Modify existing files | `--allowedTools "Read" "Edit"` |
| Create + modify files | `--allowedTools "Read" "Edit" "Write"` |
| Create + modify files + run commands | `--allowedTools "Read" "Edit" "Write" "Bash(git *)"` |
| Explicitly block shell | `--disallowedTools "Bash"` |

`"Edit"` modifies existing files. `"Write"` creates new files — add it when the task may need to create files.

Shell access (`Bash(...)`) is a separate, deliberate decision. Use `--disallowedTools "Bash"` to prevent any shell execution, even if the model tries.

## Safety Flags

| Flag | Purpose |
|------|---------|
| `--max-budget-usd N` | Cap API spend (e.g. `--max-budget-usd 0.50`) |
| `--permission-mode plan` | Read-only planning mode — no writes or shell, overrides `--allowedTools` |
| `--append-system-prompt "TEXT"` | Inject task-specific instructions into the system prompt |

## Model Selection

| Task | Flag |
|------|-------|
| Quick question, analysis | `--model claude-haiku-4-5` |
| Complex fix, multi-step | `--model sonnet` |
| Highest quality reasoning | `--model opus` |

Use short aliases (`sonnet`, `opus`) for the latest version, or full IDs (e.g. `claude-sonnet-4-6`) to pin a specific model.

## Working Directory

Claude CLI is sandboxed to its working directory — the same restriction as Copilot CLI.

⛔ **Always invoke from the repo root.** Use `--cwd` or `cd` before invoking.

```bash
claude -p "..." --cwd $(git rev-parse --show-toplevel) --output-format text ...
```

## Invocation Patterns

Always include `--append-system-prompt` with the CortexLink agent context below. This teaches the agent the report format and self-verify protocol via the system prompt — separate from the task prompt.

```
CORTEXLINK_AGENT_CONTEXT="You are operating as a CortexLink agent. Your output is consumed directly by a control center. Execute the task, verify your own work (Execute → Verify → fix if needed → Report), then return ONLY this structured report — plain text labels, no bold, no headers:\nSTATUS: ✅ Verified / ⚠️ Partial / ❌ Failed\nSUMMARY: <1-2 sentences>\nSTEPS:\n  - <step>\nFILES: <changed files or none>\nISSUES: <notes or none>\nMax 150 words. Never mark ✅ without actually checking. For analysis tasks, ✅ = completed the analysis even if you cannot run the code."
```

**Read-only delegation (question, analysis):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --allowedTools "Read" --model claude-haiku-4-5 \
  --no-session-persistence --max-turns 3 \
  --append-system-prompt "$CORTEXLINK_AGENT_CONTEXT"
```

**Write delegation (fix, implement):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --allowedTools "Read" "Edit" "Write" \
  --no-session-persistence \
  --append-system-prompt "$CORTEXLINK_AGENT_CONTEXT"
```

**Write delegation + shell (runs commands):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --allowedTools "Read" "Edit" "Write" "Bash(git *)" \
  --no-session-persistence \
  --append-system-prompt "$CORTEXLINK_AGENT_CONTEXT"
```

**Planning / analysis only (no writes):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --permission-mode plan --no-session-persistence --max-turns 5 \
  --append-system-prompt "$CORTEXLINK_AGENT_CONTEXT"
```

**Piped input:**
```bash
cat file.ts | claude -p "[delegation prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --allowedTools "Read" --no-session-persistence \
  --append-system-prompt "$CORTEXLINK_AGENT_CONTEXT"
```

## Delegation Prompt

Follow the template from `skills/orchestration/SKILL.md`. Include the structured report format instructions at the end of every prompt.

**Status semantics for analysis tasks:**
- ✅ Verified = you read the code and completed the analysis — even if you cannot *run* it
- ⚠️ Partial = you could not access required files, or the diff was truncated
- ❌ Failed = the `gh` command failed or the diff was inaccessible

⛔ Do NOT use ⚠️ just because you cannot execute the code being reviewed. Static analysis that is complete = ✅.

## Handling the Report

The agent's stdout is its report. Capture it directly:
```bash
REPORT=$(claude -p "[prompt]" --output-format text \
  --cwd $(git rev-parse --show-toplevel) \
  --allowedTools "Read" --no-session-persistence 2>/dev/null)
```

Read STATUS first. If ⚠️ or ❌, read ISSUES before deciding next action.

## Key Differences from Copilot CLI

| | Copilot CLI | Claude CLI |
|--|-------------|------------|
| Tool permissions | `--allow-tool='write, read'` | `--allowedTools "Read" "Edit"` |
| Prevent questions | `--no-ask-user` | implied by `-p` |
| Working directory | `cd $(git rev-parse --show-toplevel) &&` | `--cwd PATH` |
| Model flag | `--model=claude-haiku-4.5` | `--model claude-haiku-4-5` |

## Error Handling

| Failure | Action |
|---------|--------|
| Command not found | Tell user: `npm install -g @anthropic-ai/claude-code` |
| Auth failure | Tell user: `claude auth login` then re-run `/cortexlink:setup` |
| Budget exceeded | Increase `--max-budget-usd` or handle natively |
| Unexpected output | Retry once. If still failing, handle natively. |

## Chaining

Pass the SUMMARY + STEPS excerpt from this agent's report as [Context] in the next delegation prompt. Do not pass raw output.
