# CoFluent

> A Claude Code plugin that teaches Claude the GitHub Copilot CLI programmatic flags

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-namos2502%2FCoFluent-181717?logo=github)](https://github.com/namos2502/CoFluent)

## What it does

CoFluent bridges **Claude Code** and the **GitHub Copilot CLI**. When you ask Claude to use Copilot, it loads the CoFluent skill and knows exactly which flags to use for non-interactive, scriptable invocations — no guessing, no doc-diving.

After installing, Claude knows:

- The full set of programmatic flags (`-p`, `-s`, `--no-ask-user`, `--no-auto-update`, `--no-color`)
- How to scope tool permissions with `--allow-tool` (read, write, git, npm)
- Which model to use based on task complexity
- Ready-to-use invocation patterns for questions, suggestions, fixes, and reviews

## Requirements

- **Claude Code** v2.1.77+ — [Install guide](https://docs.anthropic.com/en/docs/claude-code)
- **GitHub Copilot CLI** — installed and authenticated

> ⚠️ The Copilot CLI (`copilot`) is **not** the same as `gh copilot`. It is a separate tool installed via Homebrew:

```bash
brew install copilot-cli   # macOS
copilot login              # authenticate once
copilot --version          # verify it's working
```

Check your Claude Code version before installing:

```bash
claude --version   # requires v1.0.33+
```

## Installation

**From GitHub (recommended):**

```bash
claude plugin install https://github.com/namos2502/CoFluent
```

**From a local clone:**

```bash
git clone https://github.com/namos2502/CoFluent
claude plugin install ./CoFluent
```

After installing, run `/reload-plugins` inside Claude Code to activate.

## Slash commands

Once installed, the following commands are available inside Claude Code:

| Command | Description |
|---------|-------------|
| `/cofluent:ask` | Ask Copilot a question (read-only, fast) |
| `/cofluent:suggest` | Get a shell command suggestion |
| `/cofluent:explain` | Explain a command, error, or snippet |
| `/cofluent:fix` | Fix a bug or error (reads and writes files) |
| `/cofluent:review` | Review staged diff or a specific file |
| `/cofluent:help` | Show command reference |

> Commands are namespaced as `/cofluent:*` when installed as a plugin. If you copy the files directly to `~/.claude/commands/`, they are available without the namespace (e.g., `/copilot-ask`, `/copilot-fix`).

## How it works

The core of CoFluent is `skills/copilot-cli/SKILL.md` — a structured reference that Claude loads automatically when working with the Copilot CLI. It provides the correct flags, tool permission syntax, and invocation patterns for programmatic use.

### Programmatic flags

| Flag | Purpose |
|------|---------|
| `-p "PROMPT"` | Non-interactive prompt — exits when done |
| `-s` | Silent mode — output only the agent response |
| `--no-ask-user` | Prevent the agent from pausing for input |
| `--no-auto-update` | Suppress update checks in scripted contexts |
| `--no-color` | Plain text output, no ANSI color codes |

Always combine these together for clean programmatic output:

```bash
copilot -p "..." -s --no-ask-user --no-auto-update --no-color
```

### Tool permissions (`--allow-tool`)

```bash
--allow-tool='read'                                        # read-only
--allow-tool='write, read'                                 # read + write files
--allow-tool='write, shell(git:*), read'                   # + git commands
--allow-tool='write, shell(git:*), shell(npm run:*), read' # + npm scripts
--allow-tool='shell(*)'                                    # full shell access
```

Use the narrowest permission set that satisfies the task.

### Model selection (`--model`)

| Task type | Recommended model |
|-----------|------------------|
| Questions, explanations, suggestions | `--model=claude-haiku-4.5` |
| Complex fixes, reviews, multi-step tasks | *(omit — defaults to Claude Opus)* |

### Common patterns

```bash
# Ask a question
copilot -p "PROMPT" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='read' --model=claude-haiku-4.5

# Fix a bug
copilot -p "Fix this error: ..." -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='write, shell(git:*), shell(npm run:*), read'

# Code review
copilot -p "/review src/index.ts" -s --no-ask-user --no-auto-update --no-color \
  --allow-tool='shell(git:*), read'
```

> Tip: Redirect stderr if you want truly clean output: `copilot -p "..." ... 2>/dev/null`

## Project structure

```
cofluent/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest
├── skills/
│   └── copilot-cli/
│       └── SKILL.md           # Core programmatic reference (auto-loaded by Claude)
└── commands/
    ├── copilot-ask.md
    ├── copilot-suggest.md
    ├── copilot-explain.md
    ├── copilot-fix.md
    ├── copilot-review.md
    └── copilot-help.md
```

## License

MIT
