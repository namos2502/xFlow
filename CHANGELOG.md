# Changelog

All notable changes to CortexLink will be documented here.

## [0.6.1] — 2026-05-04

### Fixed
- **Hooks:** `run-hook.cmd` converted from CRLF to LF line endings — eliminates `\r: command not found` errors on macOS/Linux (Windows 10/11 cmd.exe handles LF correctly)
- **Hooks:** `.gitattributes` updated to enforce `eol=lf` for `run-hook.cmd` to prevent regression on checkout

## [0.6.0] — 2026-05-01

### Added
- **Commands:** `doctor` command — replaces `auto`; symptom-first diagnostic that checks platform, plugin files, hook health, CLI agents, and setup state. Announces side effects before running; requires confirmation for live auth pings
- **Skills:** `--bare` flag added to claude-cli skill — skips CLAUDE.md, hooks, skills, MCP on startup for faster scripted delegation calls
- **Skills:** `--effort` flag added to claude-cli skill — maps to orchestration complexity levels (default for Standard, `high`/`xhigh`/`max` for Complex); Claude models only

### Improved
- **Commands:** `setup` now announces the exact block it will write before appending to CLAUDE.md / copilot-instructions.md (write preview)
- **Commands:** `help` simplified to one get-started step; commands table updated to reference `doctor`
- **Skills:** Model & Effort table in claude-cli skill aligned with orchestration complexity levels (Simple/Standard/Complex)

### Removed
- **Commands:** `auto` command removed — hook + skill auto-activation makes it redundant

### Docs
- **README:** Prerequisites section added (no version pinning)
- **README:** `[!WARNING]` restart callout added
- **README:** Troubleshooting section added (5 symptom→fix entries)
- **Hooks:** Cursor comment removed from `session-start` — Cursor is not in scope

## [0.5.7] — 2026-03-24

### Improved
- **Commands:** Added `allowed-tools` frontmatter to `setup` (Read, Edit, Bash) and `cleanup` (Read, Edit) — restricts tool availability to the minimum required set (does not suppress permission prompts; use `--allow-tool` at session level for that)
- **Commands:** Fixed `help` command directive style — replaced indirect "Reply to the user with..." with direct output instruction
- **Skills:** All three skill descriptions rewritten to third-person trigger format ("This skill should be used when...") for reliable auto-activation
- **Skills:** Added `user-invocable: false` to `orchestration`, `copilot-cli`, and `claude-cli` skills — these are internal protocol skills, not user-invocable actions
- **Skills:** Split `orchestration/SKILL.md` (8.8 KB → 5.3 KB) — delegation prompt template and report format details extracted to `references/delegation-template.md` and `references/report-format.md` for progressive disclosure

## [0.5.6] — 2026-03-23

### Fixed
- Removed `-s` flag from all Copilot CLI invocation patterns — causes exit code 1, not a valid flag for programmatic use
- Copilot CLI sandbox fix: no `--cwd` flag exists; all invocations now `cd` to repo root first (`cd $(git rev-parse --show-toplevel) &&`) — prevents silent file access failures under `--no-ask-user` when invoked from a subdirectory
- `--allow-tool` values corrected to no-space format (`'write,read'`, `'write,shell(git:*),read'`)
- Model updated to `--model=claude-sonnet-4-5` for real tasks (was "omit"); `--model=claude-haiku-4.5` kept for availability checks only
- Claude CLI: added `--cwd $(git rev-parse --show-toplevel)` to all invocation patterns — same sandbox restriction applies
- Orchestration detection command: removed `-s`, added `cd` to repo root

## [0.5.5] — 2026-03-23

### Changed
- Renamed project from **xFlow** to **CortexLink**
- All commands renamed from `/xflow:*` to `/cortexlink:*`
- Plugin install slug changed from `xflow@agent-plugins` to `cortexlink@agent-plugins`
- GitHub repo to be renamed from `namos2502/xFlow` to `namos2502/CortexLink`

## [0.5.4] — 2026-03-21

### Added
- Task complexity scale (Simple / Standard / Complex) in orchestration skill — classifies tasks before routing to determine spec detail and delegation style
- Effort threshold: Simple tasks stay inline or use native subagents — cross-CLI overhead is not justified for single-operation or read-only tasks
- Q&A phase for Complex tasks — control center appends a 1-turn question block to the delegation prompt before execution; agent surfaces ambiguity, control center refines, then execution proceeds
- Updated decision tree — checks task complexity before type; native subagent tier made explicit

## [0.5.3] — 2026-03-21

### Fixed
- `/cortexlink:auto` guard restructured from a soft "Step 1" bullet to a hard `⛔ STOP` block — models were reading past the conditional and activating regardless; now the gate is unambiguous and precedes the activation section entirely
- `/cortexlink:setup` now ends every confirmation with a prompt to run `/cortexlink:auto` next — closes the gap where users had no clear signal of what to do after setup

## [0.5.2] — 2026-03-21

### Added
- `/cortexlink:auto` now checks both `~/.claude/CLAUDE.md` and `~/.copilot/copilot-instructions.md` for the `## CortexLink` section before activating — blocks if neither is set up, warns if only one is present

### Fixed
- `/cortexlink:auto` no longer activates silently when setup hasn't been run

## [0.5.1] — 2026-03-21

### Added
- `/cortexlink:cleanup` — removes the `## CortexLink` sections from `~/.claude/CLAUDE.md` and `~/.copilot/copilot-instructions.md` added during setup

### Changed
- `/cortexlink:help` — simplified to plugin info and getting started guide; no longer shows internal skill structure

### Fixed
- `/cortexlink:help` — removed stray `---` from command body that was being parsed as a second frontmatter block, causing Claude to show its default greeting instead of the help content

## [0.5.0-beta] — 2026-03-21

### Changed — Breaking
- **Architecture:** Complete redesign from passive flag-reference plugin to active control center orchestration system
- **Skills restructured:** `skills/copilot-cli/` and `skills/claude-cli/` replaced by `skills/orchestration/` (control center protocol) and `skills/agents/copilot-cli/` + `skills/agents/claude-cli/` (behavioral agent skills)
- **Commands trimmed:** Removed `ask`, `fix`, `explain`, `suggest`, `review` — the orchestration skill handles routing automatically. Kept `setup`, `auto`, `help`.
- **`/cortexlink:auto`** repurposed: from "primary activation" to "explicit session override" — after `/cortexlink:setup`, CortexLink is always-on via `~/.claude/CLAUDE.md`
- **`/cortexlink:setup`** now registers CortexLink in both `~/.claude/CLAUDE.md` and `~/.copilot/copilot-instructions.md` — always-on in Claude Code and Copilot CLI

### Added
- `skills/orchestration/SKILL.md` — control center protocol: routing decision tree, delegation prompt template, structured report format, self-verify loop, error handling, cross-agent chaining, recursion guard
- `skills/agents/copilot-cli/SKILL.md` — behavioral reference: when to use, flags, invocation patterns
- `skills/agents/claude-cli/SKILL.md` — behavioral reference: when to use, flags, model selection, key differences from Copilot CLI
- **Control Center model** — your active AI agent acts as the director; fans out tasks to peer CLI agents, reviews every report before proceeding
- **Agent model** — each CLI agent is a full peer, not a dumb executor; uses its own platform-native tools and sub-agents internally, self-verifies, and reports back
- **Fan-out tree** — one level deep, width scales; agents don't chain to each other
- **Structured report format** — every agent returns `STATUS / SUMMARY / STEPS / FILES / ISSUES` (~150 words)
- **Self-verify before reporting** — agents must verify their own work before returning ✅
- **Extensible agent architecture** — `skills/agents/<name>/SKILL.md` pattern; drop a folder to add a new agent

---

## [0.4.0-beta] — 2026-03-19

### Added
- `skills/claude-cli/SKILL.md` — programmatic reference for the Claude CLI; covers `claude -p` flags, tool permission syntax (`--allowedTools`, `--disallowedTools`, `--tools`), model selection, budget caps, and common invocation patterns
- `/cortexlink:auto` now loads both `copilot-cli` and `claude-cli` skill references; includes routing guidance (Copilot for GitHub tasks, Claude for general code tasks)
- `/cortexlink:setup` now detects and authenticates both Copilot CLI and Claude CLI; CLAUDE.md block updated to describe both agents
- `/cortexlink:help` now shows commands in a multi-agent context

---

## [0.3.0-beta] — 2026-03-19

### Changed
- Renamed project from **CoFluent** to **CortexLink** — reflecting the broader multi-agent vision
- New tagline: "One plugin. Every CLI. Stay in flow."
- All commands renamed from `/cofluent:*` to `/cortexlink:*`
- Plugin install slug changed from `cofluent@agent-plugins` to `cortexlink@agent-plugins`
- GitHub repo renamed from `namos2502/CoFluent` to `namos2502/CortexLink`

---

## [0.2.0-beta] — 2026-03-18

### Added
- `/cortexlink:setup` — one-time setup command that verifies Copilot CLI is installed and authenticated, then registers CortexLink awareness in `~/.claude/CLAUDE.md` so Claude knows about CortexLink in every future session (idempotent — safe to re-run)
- `## Why CortexLink` section in README covering two key benefits: staying in flow and saving tokens on Claude by offloading subtasks to a separate `copilot -p` process

### Removed
- `/cortexlink:verify` — absorbed into `/cortexlink:setup`; re-run setup anytime to re-check your installation


---
## [0.1.0-beta] — 2026-03-18

Initial beta release. Core plugin structure is stable; commands and skill reference are functional but may evolve based on feedback.

### Added
- `skills/copilot-cli/SKILL.md` — core programmatic reference for the GitHub Copilot CLI; auto-loaded by Claude when invoking `copilot -p`
- `/cortexlink:auto` — activates autonomous Copilot CLI mode for the session; Claude will invoke `copilot -p` automatically when appropriate
- `/cortexlink:verify` — checks that Copilot CLI is installed and authenticated; guides the user through fixing any issues
- `/cortexlink:ask` — delegates a question to Copilot (read-only, fast, uses `claude-haiku-4.5`)
- `/cortexlink:suggest` — asks Copilot to suggest a shell command for a task
- `/cortexlink:explain` — asks Copilot to explain a command, error, or code snippet
- `/cortexlink:fix` — asks Copilot to fix a bug or error (read + write file permissions)
- `/cortexlink:review` — runs the Copilot `/review` agent on staged diff or a specific file
- `/cortexlink:help` — shows the full command reference
- `.claude-plugin/plugin.json` — plugin manifest for installation via Claude Code plugin system
- Marketplace listing in [namos2502/agent-plugins](https://github.com/namos2502/agent-plugins)

### Known limitations
- Requires GitHub Copilot CLI (`brew install copilot-cli`) — separate from `gh copilot`
- Requires active Copilot subscription and `copilot login` before use
- Commands are namespaced as `/cortexlink:*` when installed as a plugin
