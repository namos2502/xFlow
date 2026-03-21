# xFlow

> One plugin. Every CLI. Stay in flow.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-namos2502%2FxFlow-181717?logo=github)](https://github.com/namos2502/xFlow)

You know the feeling. You are deep in a vibe coding session — ideas flowing, your agent keeping up — and then you need to hand something off to another CLI tool. Suddenly you are out of the zone, hunting for the right flags, checking docs, figuring out what `--allow-tool` even accepts. The flow breaks.

xFlow fixes that.

Introducing **xFlow** — a cross-agent workflow plugin for your AI agent's CLI. It gives your agent the full programmatic reference for every CLI tool you use — flags, permissions, patterns, model choices — baked right in. Install it once, and your agent already knows how to speak Copilot, Claude CLI, and more. No interruptions, no setup tax, no context switching. You stay in the zone and the handoff happens automatically.

The way it should work.

## Why xFlow

**Stay in flow.** No context switching, no flag hunting, no setup tax. Your agent already knows the programmatic reference for every supported CLI — baked right in.

**Reduce your token spend.** Every delegated task runs in a separate subprocess — only the final answer comes back into context. No reasoning chains, no intermediate tool calls. Your context stays lean and your quota goes further.

## Get started

Before installing, make sure you have at least one supported CLI agent set up.

**GitHub Copilot CLI:**
```bash
brew install copilot-cli
copilot login
```

**Claude CLI:**
```bash
npm install -g @anthropic-ai/claude-code
claude auth login
```

Then install xFlow from inside Claude Code:

```
/plugin marketplace add namos2502/agent-plugins
/plugin install xflow@agent-plugins
/reload-plugins
```

Then run setup once:

```
/xflow:setup
```

This detects which CLI agents are installed, verifies authentication, and registers xFlow awareness in your `~/.claude/CLAUDE.md` so it's available in every future session.

Requires Claude Code v1.0.33+.

## Commands

| Command | What it does |
|---------|-------------|
| `/xflow:setup` | One-time setup — detects installed agents, authenticates, and registers xFlow in your `~/.claude/CLAUDE.md` |
| `/xflow:auto` | Activates multi-agent mode — loads all CLI references and routes tasks automatically |
| `/xflow:ask` | Delegates a question to the best available agent |
| `/xflow:suggest` | Gets a shell command suggestion |
| `/xflow:explain` | Explains a command, error, or snippet |
| `/xflow:fix` | Fixes a bug or error (reads and writes files) |
| `/xflow:review` | Reviews staged diff or a specific file |
| `/xflow:help` | Shows this reference |

### Pick your style

**Hands-off** — run `/xflow:auto` once and let your agent decide when to reach for which CLI. Good for longer sessions where you just want things to work.

**Hands-on** — use individual commands to point your agent at a specific task. You stay in control of every delegation.

## Under the hood

xFlow uses a skill-per-agent architecture. Each supported CLI tool ships its own skill file loaded as a programmatic reference:

- `skills/copilot-cli/SKILL.md` — GitHub Copilot CLI reference
- `skills/claude-cli/SKILL.md` — Claude CLI reference

`/xflow:auto` loads all available skill files for the session, giving your agent the full picture of every CLI at its disposal.

The base Copilot pattern:
```bash
copilot -p "..." -s --no-ask-user --no-auto-update --no-color --allow-tool='read'
```

The base Claude pattern:
```bash
claude -p "..." --output-format text --allowedTools "Read"
```

## Supported agents

| CLI         | CMD          | Status       |
|-------------|--------------|--------------|
| Copilot CLI | `copilot -p` | ✅ Supported |
| Claude Code | `claude -p`  | ✅ Supported |

## License

MIT
