---
name: copilot-cli
description: Behavioral reference for GitHub Copilot CLI. Use when delegating GitHub-specific or GitHub-adjacent tasks via `copilot -p`.

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
| `-s` | Silent — output only the agent response (no usage stats) |
| `--no-ask-user` | Agent works autonomously, no questions |
| `--no-auto-update` | Suppress update checks |
| `--no-color` | Plain text output |

Always combine `-s --no-ask-user --no-auto-update --no-color` for clean programmatic output.

## Tool Permissions (`--allow-tool`)

Pre-approves tools so the agent doesn't pause to prompt.

| Use case | Flag |
|----------|------|
| Questions, analysis, review | `--allow-tool='read'` |
| Modify or create files | `--allow-tool='write, read'` |
| Modify files + run git commands | `--allow-tool='write, shell(git:*), read'` |

Shell access (`shell(...)`) is a separate, deliberate decision — not an automatic addition to write access. Only grant it when the task genuinely requires running commands.

Use `--deny-tool` to block specific commands within an allowed scope:
```bash
# Allow git reads but block pushes
--allow-tool='shell(git:*), read' --deny-tool='shell(git push)'
```
Deny rules always override allow rules.

## Model Selection

| Task | Flag |
|------|-------|
| Quick question, analysis | `--model=claude-haiku-4.5` |
| Complex fix, multi-step | omit (uses session default) |

## Invocation Patterns

**Read-only delegation (question, analysis):**
```bash
copilot -p "[delegation prompt]" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='read' --model=claude-haiku-4.5
```

**Write delegation (fix, implement):**
```bash
copilot -p "[delegation prompt]" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='write, read'
```

**Write delegation + git (runs git commands):**
```bash
copilot -p "[delegation prompt]" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='write, shell(git:*), read'
```

**Code review (built-in /review agent):**
```bash
copilot -p "/review [scope]" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='shell(git:*), read'
```

Redirect stderr if needed: add `2>/dev/null`

## Delegation Prompt

Follow the template from `skills/orchestration/SKILL.md`. Include the structured report format instructions at the end of every prompt.

⛔ **Critical:** Always include this exact line in your delegation prompt:
*"Return ONLY the structured report. No reasoning steps, no 'Let me...' output before the report."*

## Handling the Report

The agent's stdout is its report. Capture it directly:
```bash
REPORT=$(copilot -p "[prompt]" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' 2>/dev/null)
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
