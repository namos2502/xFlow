---
name: claude-cli
description: Programmatic reference for Claude CLI. Use this when invoking `claude -p` to delegate tasks non-interactively.
---

# Claude CLI — Programmatic Reference

When delegating tasks to the Claude CLI, always invoke it via `claude -p` with the correct flags for non-interactive, scriptable use.

## Core programmatic flags

| Flag | Purpose |
|------|---------|
| `-p "PROMPT"` | Print mode — pass a prompt directly, exits when done (enables non-interactive mode) |
| `--output-format text` | Plain text output (default, easiest to read back) |
| `--output-format json` | Structured JSON output with metadata |
| `--output-format stream-json` | Streaming JSON events |
| `--no-session-persistence` | Do not save this session to disk (keeps things clean for scripted calls) |

## Tool permissions

Control which tools the Claude agent can use during the delegated task.

### `--allowedTools` — whitelist specific tools (execute without prompting)

| Use case | Flag value |
|----------|-----------|
| Read-only (questions, explanations) | `--allowedTools "Read"` |
| Read + write files | `--allowedTools "Read" "Edit" "Write"` |
| Read + write + git commands | `--allowedTools "Read" "Edit" "Bash(git *)"` |
| Read + write + git + npm scripts | `--allowedTools "Read" "Edit" "Bash(git *)" "Bash(npm run:*)"` |

### `--disallowedTools` — blacklist tools entirely

| Use case | Flag value |
|----------|-----------|
| Prevent all shell execution | `--disallowedTools "Bash"` |
| Prevent file writes | `--disallowedTools "Edit" "Write"` |

## Model selection (`--model`)

| Task type | Recommended model |
|-----------|------------------|
| Quick questions, explanations, suggestions | `--model claude-haiku-4-5` |
| Complex fixes, reviews, multi-step tasks | `--model sonnet` (latest Sonnet) |
| Highest quality reasoning | `--model opus` (latest Opus) |

Use the short aliases `sonnet` and `opus` for the latest versions, or full model IDs (e.g. `claude-sonnet-4-6`) for pinned versions.

## System prompt flags

| Flag | Purpose |
|------|---------|
| `--append-system-prompt "TEXT"` | Append custom instructions to the default system prompt — use to inject task-specific context without replacing built-in behavior |

## Budget and safety flags

| Flag | Purpose |
|------|---------|
| `--max-turns N` | Limit the number of agentic turns — prevents runaway tasks (print mode only) |
| `--max-budget-usd N` | Cap API spending for this call (print mode only, e.g. `--max-budget-usd 0.50`) |
| `--fallback-model MODEL` | Fall back to this model if the primary is overloaded (print mode only) |
| `--permission-mode plan` | Read-only planning mode — Claude can read and plan but cannot write files or run commands. Good safety default for analysis tasks |

## Common invocation patterns

**Ask a question (read-only, fast):**
```bash
claude -p "PROMPT" --output-format text --allowedTools "Read" --model claude-haiku-4-5 --no-session-persistence --max-turns 3
```

**Suggest a shell command:**
```bash
claude -p "Suggest a shell command to: TASK" --output-format text --allowedTools "Read" --model claude-haiku-4-5 --no-session-persistence --max-turns 2
```

**Explain code or a command:**
```bash
claude -p "Explain this: CONTENT" --output-format text --allowedTools "Read" --model claude-haiku-4-5 --no-session-persistence --max-turns 3
```

**Fix a bug or error (can write files):**
```bash
claude -p "Fix this error: ERROR\n\nContext: CONTEXT" --output-format text --allowedTools "Read" "Edit" "Bash(git *)" --no-session-persistence
```

**Review code (read-only):**
```bash
claude -p "Review this code for bugs and issues: CONTENT" --output-format text --allowedTools "Read" "Bash(git diff *)" "Bash(git log *)" --no-session-persistence
```

**Process piped content:**
```bash
cat file.ts | claude -p "Explain this code" --output-format text --allowedTools "Read" --no-session-persistence
```

## Key differences from Copilot CLI

| | Copilot CLI | Claude CLI |
|---|-------------|------------|
| Print mode flag | `-p` | `-p` |
| Silence/suppress decoration | `-s` | `--output-format text` |
| Prevent questions | `--no-ask-user` | implied by `-p` |
| Suppress updates | `--no-auto-update` | implied (use `--no-session-persistence`) |
| Tool permissions | `--allow-tool='write, read'` | `--allowedTools "Read" "Edit"` |
| Model selection | `--model=claude-haiku-4-5` | `--model claude-haiku-4-5` |
