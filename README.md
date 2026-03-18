# CoFluent

> A Claude Code plugin that teaches Claude the GitHub Copilot CLI — so it can invoke `copilot -p` correctly without guessing.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-namos2502%2FCoFluent-181717?logo=github)](https://github.com/namos2502/CoFluent)

## Install

```
/plugin marketplace add namos2502/agent-plugins
/plugin install cofluent@agent-plugins
/reload-plugins
```

> Requires Claude Code v1.0.33+ and [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli) (`brew install copilot-cli` + `copilot login`).
>
> The Copilot CLI (`copilot`) is **not** the same as `gh copilot` — it is a separate tool.

## Commands

| Command | Description |
|---------|-------------|
| `/cofluent:auto` | Load Copilot CLI reference for the session — Claude will invoke `copilot -p` autonomously when appropriate |
| `/cofluent:verify` | Check Copilot CLI is installed and authenticated |
| `/cofluent:ask` | Ask Copilot a question |
| `/cofluent:suggest` | Get a shell command for a task |
| `/cofluent:explain` | Explain a command, error, or snippet |
| `/cofluent:fix` | Fix a bug or error (can read and write files) |
| `/cofluent:review` | Review staged diff or a specific file |
| `/cofluent:help` | Show this reference |

### Two modes

**Autonomous** — run `/cofluent:auto` once. Claude will use `copilot -p` on its own throughout the session.

**Explicit** — use individual commands to delegate a specific task directly.

## How it works

The core of CoFluent is `skills/copilot-cli/SKILL.md` — a reference Claude loads to know the correct [programmatic flags](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-programmatic-reference), tool permissions, and invocation patterns for non-interactive use.

```bash
# The base pattern Claude always uses
copilot -p "..." -s --no-ask-user --no-auto-update --no-color --allow-tool='read'
```

## License

MIT
