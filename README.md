# CortexLink

> One plugin. Every CLI. Stay in flow.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-namos2502%2FCortexLink-181717?logo=github)](https://github.com/namos2502/CortexLink)

You know the feeling. You are deep in a vibe coding session — ideas flowing, your agent keeping up — and then you need to hand something off to another CLI tool. Suddenly you are out of the zone, hunting for the right flags, checking docs, figuring out what `--allow-tool` even accepts. The flow breaks.

CortexLink fixes that.

**CortexLink** is a cross-agent orchestration plugin for your AI agent CLI. It turns your active AI agent into a persistent **control center** that decomposes tasks, delegates to other CLI agents, and manages the full workflow — while you stay focused on what you're building.

> **Note:** CortexLink is designed for use with 2 or more AI CLI platforms. To get the most out of cross-agent workflows, you need access to at least two supported CLI agents (e.g. Copilot CLI + Claude Code).

## How it works

CortexLink uses a **Control Center → Agents** model:

```
CONTROL CENTER (your AI agent — decomposes, routes, reviews)
  ├── AGENT (Copilot CLI)
  ├── AGENT (Claude CLI)
  └── AGENT (future-cli ...)
        └── [own tools & sub-agents — internal, platform-native]
```

- The **control center** is your active AI agent. It holds the plan, fans out tasks to the right CLI agents (in parallel or in sequence), and reviews every report before proceeding.
- Each **agent** is a full CLI agent in its own right — not just an executor. It can use its own platform-native tools and sub-agents internally, then self-verifies and reports back.
- The tree is **one level deep**. Agents are peers — they don't chain to each other. Width scales as you add agents; depth stays fixed.
- Every agent returns a **structured report** (~150 words: status, steps, files changed, issues)

No context bloat. No manual coordination. The workflow just happens.

## Why CortexLink

**Stay in flow.** Your active CLI knows how to route tasks, which agent to use, and how to handle the results — without you managing it.

**Reduce your token spend.** Every delegated subtask runs in a separate subprocess. Only the final report comes back into your context — no reasoning chains, no intermediate tool calls.

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

Then install CortexLink from inside Claude Code:

```
/plugin marketplace add namos2502/agent-plugins
/plugin install cortexlink@agent-plugins
/reload-plugins
```

> **Note:** If commands don't appear after `/reload-plugins`, restart Claude Code completely — a full restart is sometimes needed to pick up new commands.

Then run setup once:

```
/cortexlink:setup
```

This detects which CLI agents are installed, verifies authentication, and registers CortexLink in your `~/.claude/CLAUDE.md` so it's active in every future session — no need to run anything again.

## Updating

To get the latest version:

```
/plugin marketplace update agent-plugins
/plugin update cortexlink@agent-plugins
```

Then restart Claude Code to pick up any new or changed commands.

## Uninstalling

To remove CortexLink completely:

1. Run `/cortexlink:cleanup` — removes the CortexLink sections added to `~/.claude/CLAUDE.md` and `~/.copilot/copilot-instructions.md`
2. Run `/plugin uninstall cortexlink@agent-plugins` — removes the plugin files

Requires Claude Code v1.0.33+.

## Commands

| Command | What it does |
|---------|-------------|
| `/cortexlink:setup` | One-time setup — detects agents, authenticates, registers CortexLink as always-on in `~/.claude/CLAUDE.md` |
| `/cortexlink:auto` | Explicitly load all skills and activate orchestration mode for this session |
| `/cortexlink:cleanup` | Remove CortexLink configuration added by setup |
| `/cortexlink:help` | Show plugin information and commands |

## Under the hood

CortexLink uses a skill-per-concern architecture:

```
skills/
  orchestration/
    SKILL.md          ← control center protocol, routing decision tree, report format
  agents/
    copilot-cli/
      SKILL.md        ← Copilot CLI behavioral reference
    claude-cli/
      SKILL.md        ← Claude CLI behavioral reference
    [future-agent]/   ← drop here to add a new agent
      SKILL.md
```

After `/cortexlink:setup`, the `~/.claude/CLAUDE.md` block makes CortexLink always-on — the orchestration protocol activates automatically every session. `/cortexlink:auto` is available as an explicit override if needed.

## Routing

The orchestration skill teaches your CLI when and how to delegate:

| Task type | Route to |
|-----------|---------|
| GitHub-specific (PRs, repos, Actions) | Copilot CLI |
| General code tasks, Anthropic-native reasoning | Claude CLI |
| Needs current session context or open files | Handle natively |
| Host has a native subagent for it | Use it (faster, no auth needed) |

## Supported agents

| CLI         | Cmd          | Status       |
|-------------|--------------|--------------|
| Copilot CLI | `copilot -p` | ✅ Supported |
| Claude Code | `claude -p`  | ✅ Supported |

## License

This project is licensed under the MIT License
