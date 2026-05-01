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

**Reduce your token spend.** Every delegated subtask runs in a separate subprocess. Only the final result comes back into your context — no reasoning chains, no intermediate tool calls.

## Prerequisites

- **Claude Code** — CortexLink is a Claude Code plugin
- **At least one supported CLI agent** installed and authenticated (see Get started below)

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

> [!WARNING]
> If commands don't appear after `/reload-plugins`, fully quit Claude Code (Cmd+Q on macOS, close the window on Windows) and relaunch. `/reload-plugins` does not always pick up new commands without a full restart.

Then run setup once:

```
/cortexlink:setup
```

This detects which CLI agents are installed, verifies authentication, and activates CortexLink — the `SessionStart` hook will inject orchestration context automatically on every future session.

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

## Troubleshooting

**Commands don't appear after `/reload-plugins`**
Fully quit Claude Code (Cmd+Q on macOS, close the window on Windows) and relaunch. `/reload-plugins` does not always pick up new commands without a full restart.

**The SessionStart hook doesn't seem to be injecting context**
Run `/cortexlink:doctor` — it checks the hook script, verifies `jq` is installed, and test-runs the hook to confirm it produces valid output.

**Setup reports an agent as not found but it's installed**
The agent binary may not be on Claude Code's PATH. Open a terminal and run `which copilot` or `which claude`. If it's found there but not in setup, your shell PATH may differ from Claude Code's environment — check your shell config (`~/.zshrc`, `~/.bashrc`).

**Auth check fails but I'm already logged in**
Re-run the login command (`copilot login` or `claude auth login`). Plugin sandboxing can sometimes make cached tokens appear invalid — a fresh login usually fixes it.

**Delegation fails or the agent doesn't respond**
Run `/cortexlink:doctor` to check agent auth and hook status. Verify the agent binary is on PATH (see above). Make sure the agent isn't already running interactively in another terminal.


| Command | What it does |
|---------|-------------|
| `/cortexlink:setup` | One-time setup — detects agents, authenticates, registers CortexLink as always-on in `~/.claude/CLAUDE.md` |
| `/cortexlink:doctor` | Diagnose the SessionStart hook, plugin files, and agent auth — asks about symptoms before running checks |
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

After `/cortexlink:setup`, the `SessionStart` hook injects the orchestration protocol automatically on every new session — no extra command needed. Run `/cortexlink:doctor` if anything seems off.

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
