---
name: claude-cli
description: Behavioral reference for Claude CLI. Use when delegating general code tasks, analysis, or tasks requiring Anthropic-native reasoning via `claude -p`.

---

# Claude CLI Agent

## When to Use

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

## Invocation Patterns

**Read-only delegation (question, analysis):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --allowedTools "Read" --model claude-haiku-4-5 \
  --no-session-persistence --max-turns 3
```

**Write delegation (fix, implement):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --allowedTools "Read" "Edit" "Write" \
  --no-session-persistence
```

**Write delegation + shell (runs commands):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --allowedTools "Read" "Edit" "Bash(git *)" \
  --no-session-persistence
```

**Planning / analysis only (no writes):**
```bash
claude -p "[delegation prompt]" --output-format text \
  --permission-mode plan --no-session-persistence --max-turns 5
```

**Piped input:**
```bash
cat file.ts | claude -p "[delegation prompt]" --output-format text \
  --allowedTools "Read" --no-session-persistence
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
REPORT=$(claude -p "[prompt]" --output-format text --allowedTools "Read" --no-session-persistence 2>/dev/null)
```

Read STATUS first. If ⚠️ or ❌, read ISSUES before deciding next action.

## Key Differences from Copilot CLI

| | Copilot CLI | Claude CLI |
|--|-------------|------------|
| Tool permissions | `--allow-tool='write, read'` | `--allowedTools "Read" "Edit"` |
| Silence | `-s` | `--output-format text` |
| Prevent questions | `--no-ask-user` | implied by `-p` |
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
