---
name: copilot-cli
description: Programmatic reference for GitHub Copilot CLI. Use this when invoking `copilot -p` to delegate tasks non-interactively.
---

# GitHub Copilot CLI — Programmatic Reference

When delegating tasks to the GitHub Copilot CLI, always invoke it via `copilot -p` with the correct flags for non-interactive, scriptable use.

## Core programmatic flags

| Flag | Purpose |
|------|---------|
| `-p "PROMPT"` | Pass a prompt directly — enables non-interactive mode, exits when done |
| `-s` | Silent mode — suppress stats and decoration, output only the agent response |
| `--no-ask-user` | Prevent the agent from pausing to ask the user questions |
| `--no-auto-update` | Suppress update checks (important in scripted/automated contexts) |
| `--no-color` | Plain text output, no ANSI color codes |

## Tool permissions (`--allow-tool`)

Control which tools the Copilot agent can use. Syntax: comma-separated list in quotes.

| Use case | Flag value |
|----------|-----------|
| Read-only (questions, explanations) | `--allow-tool='read'` |
| Read + write files | `--allow-tool='write, read'` |
| Read + write + git commands | `--allow-tool='write, shell(git:*), read'` |
| Read + write + git + npm scripts | `--allow-tool='write, shell(git:*), shell(npm run:*), read'` |
| Full shell access | `--allow-tool='shell(*)'` |

Pattern syntax: `shell(command:subcommand)` or `shell(command:*)` for all subcommands of a tool.

## Model selection (`--model`)

| Task type | Recommended model |
|-----------|------------------|
| Quick questions, explanations, suggestions | `--model=claude-haiku-4.5` |
| Complex fixes, reviews, multi-step tasks | omit flag (defaults to Claude Opus) |

## Common invocation patterns

**Ask a question (read-only, fast):**
```bash
copilot -p "PROMPT" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=claude-haiku-4.5
```

**Suggest a shell command:**
```bash
copilot -p "Suggest a shell command to: TASK" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=claude-haiku-4.5
```

**Explain code or a command:**
```bash
copilot -p "Explain this: CONTENT" -s --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=claude-haiku-4.5
```

**Fix a bug or error (can write files):**
```bash
copilot -p "Fix this error: ERROR\n\nContext: CONTEXT" -s --no-ask-user --no-auto-update --no-color --allow-tool='write, shell(git:*), shell(npm run:*), read'
```

**Code review (uses built-in `/review` agent):**
```bash
copilot -p "/review SCOPE" -s --no-ask-user --no-auto-update --no-color --allow-tool='shell(git:*), read'
```

## Notes

- Always combine `-s`, `--no-ask-user`, `--no-auto-update`, and `--no-color` together for clean programmatic output.
- Redirect stderr if needed: `copilot -p "..." ... 2>/dev/null`
- Use `--allow-tool` as narrowly as possible — prefer `read` only unless writes are truly needed.
