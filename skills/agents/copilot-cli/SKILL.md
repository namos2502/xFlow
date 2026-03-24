---
name: copilot-cli
description: This skill should be used when delegating GitHub-specific tasks — PRs, issues, repos, Actions, branches, code review — via `copilot -p`. Use it when Copilot CLI's GitHub-native tooling is an advantage over the host agent.
user-invocable: false
---

# Copilot CLI Agent

## When to Use

**Use Copilot CLI for:**
- GitHub-specific tasks — PRs, issues, repos, Actions, branches, code review
- Git workflow tasks where GitHub context helps
- Tasks where Copilot's GitHub-native tooling is an advantage

**Do NOT use Copilot CLI for:**
- General code explanations or refactors (handle natively or use Claude CLI)
- Tasks that need your current session context
- When Copilot is not installed or not authenticated

## Core Flags

| Flag | Purpose |
|------|---------|
| `-p "PROMPT"` | Programmatic mode — executes prompt and exits |
| `--no-ask-user` | Suppresses interactive permission prompts — required for non-interactive use |
| `--no-auto-update` | Prevents Copilot from updating itself mid-run |
| `--no-color` | Clean output, no ANSI escape codes |
| `-s` | ❌ Causes exit code 1 — do not use |

Always combine `--no-ask-user --no-auto-update --no-color` for clean programmatic output.

## Sandbox Limitation

Copilot CLI is sandboxed to its working directory. With `--no-ask-user`, any file access outside that directory silently fails — no error, no output. There is no `--cwd` flag; the only fix is to `cd` to the repo root before invoking.

⛔ **Always `cd` to the repo root before invoking.** Never call from a subdirectory.

```bash
# ❌ Sandboxed to client/ — reads outside it silently fail
copilot -p "..." --no-ask-user ...

# ✅ Correct — full repo accessible
cd $(git rev-parse --show-toplevel) && copilot -p "..." --no-ask-user ...
```

## Tool Permissions (`--allow-tool`)

Pre-approves tools so the agent doesn't pause to prompt.

| Use case | Flag |
|----------|------|
| Questions, analysis, review | `--allow-tool='read'` |
| Modify or create files | `--allow-tool='write,read'` |
| Modify files + run git commands | `--allow-tool='write,shell(git:*),read'` |

Shell access (`shell(...)`) is a separate, deliberate decision — not an automatic addition to write access. Only grant it when the task genuinely requires running commands.

Use `--deny-tool` to block specific commands within an allowed scope:
```bash
# Allow git reads but block pushes
--allow-tool='shell(git:*),read' --deny-tool='shell(git push)'
```
Deny rules always override allow rules.

## Model Selection

| Task | Flag |
|------|-------|
| Availability check / quick question | `--model=claude-haiku-4.5` |
| Real tasks — analysis, fix, review | `--model=claude-sonnet-4-5` |

## Invocation Patterns

**Read-only delegation (question, analysis):**
```bash
cd $(git rev-parse --show-toplevel) && \
copilot -p "[delegation prompt]" --no-ask-user --no-auto-update --no-color \
  --allow-tool='read' --model=claude-sonnet-4-5
```

**Write delegation (fix, implement):**
```bash
cd $(git rev-parse --show-toplevel) && \
copilot -p "[delegation prompt]" --no-ask-user --no-auto-update --no-color \
  --allow-tool='write,read' --model=claude-sonnet-4-5
```

**Write delegation + git (runs git commands):**
```bash
cd $(git rev-parse --show-toplevel) && \
copilot -p "[delegation prompt]" --no-ask-user --no-auto-update --no-color \
  --allow-tool='write,shell(git:*),read' --model=claude-sonnet-4-5
```

**Code review (built-in /review agent):**
```bash
cd $(git rev-parse --show-toplevel) && \
copilot -p "/review [scope]" --no-ask-user --no-auto-update --no-color \
  --allow-tool='shell(git:*),read' --model=claude-sonnet-4-5
```

Redirect stderr if needed: add `2>/dev/null`

## Delegation Prompt

Follow the template from `skills/orchestration/SKILL.md`. Include the structured report format instructions at the end of every prompt.

⛔ **Critical:** Always include this exact line in your delegation prompt:
*"Return ONLY the structured report. No reasoning steps, no 'Let me...' output before the report."*

## Handling the Report

The agent's stdout is its report. Capture it directly:
```bash
REPORT=$(cd $(git rev-parse --show-toplevel) && \
  copilot -p "[prompt]" --no-ask-user --no-auto-update --no-color \
  --allow-tool='read' --model=claude-sonnet-4-5 2>/dev/null)
```

Read STATUS first. If ⚠️ or ❌, read ISSUES before deciding next action.

## Error Handling

| Failure | Action |
|---------|--------|
| Command not found | Tell user: `brew install copilot-cli` |
| Auth failure / no output | Tell user: `copilot login` then re-run `/cortexlink:setup` |
| Unexpected output format | Retry once. If still malformed, handle the task natively. |

## Chaining

Pass the SUMMARY + STEPS excerpt from this agent's report as [Context] in the next delegation prompt. Do not pass raw output.
