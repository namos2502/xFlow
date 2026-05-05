---
description: "Diagnose CortexLink issues — asks about symptoms, runs targeted checks, and confirms before fixing anything"
allowed-tools: Read, Bash
---

⛔ **BEFORE ANYTHING ELSE — verify the environment:**

```bash
echo "${CLAUDE_PLUGIN_ROOT:-NOT_SET}"
```

If the output is `NOT_SET`:
> **STOP. Do not proceed.**
> Tell the user: ⚠️ `CLAUDE_PLUGIN_ROOT` is not set. Run this command inside Claude Code or Copilot CLI with the CortexLink plugin loaded.

---

**Ask the user:**

> What are you experiencing? Describe what's happening and I'll run the right checks.
>
> Examples:
> - "The hook doesn't seem to fire / I don't see orchestration context injected"
> - "Commands aren't showing up after install"
> - "Agent delegation isn't working or auth is failing"
> - "Setup seems incomplete or broken"
> - "full checkup" — run everything

Wait for the user's response before continuing.

---

Map their symptom(s) to the checks below and run only those — unless they said "full checkup", in which case run all five.

| Symptom | Checks to run |
|---------|--------------|
| Hook not firing / no orchestration context | 1, 3 |
| Commands missing / plugin not loading | 1, 2 |
| Delegation failing / auth issues | 1, 4 |
| Setup incomplete or broken | 1, 5 |
| Full checkup | 1, 2, 3, 4, 5 |

**Rule for every check:** report the finding first. If a fix is possible, tell the user exactly what you found and what you're about to do — then ask for confirmation before doing it.

---

## Check 1 — Platform & Environment

```bash
echo "CLAUDE_PLUGIN_ROOT=${CLAUDE_PLUGIN_ROOT}"
echo "COPILOT_CLI=${COPILOT_CLI:-not set}"
```

- `CLAUDE_PLUGIN_ROOT` set, `COPILOT_CLI` not set → **Claude Code**
- Both set → **Copilot CLI**
- Report the detected platform. Note it for the summary.

---

## Check 2 — Plugin File Integrity

```bash
for f in \
  "$CLAUDE_PLUGIN_ROOT/hooks/hooks.json" \
  "$CLAUDE_PLUGIN_ROOT/hooks/run-hook.cmd" \
  "$CLAUDE_PLUGIN_ROOT/hooks/session-start" \
  "$CLAUDE_PLUGIN_ROOT/skills/orchestration/SKILL.md" \
  "$CLAUDE_PLUGIN_ROOT/skills/orchestration/references/agent-context.md" \
  "$CLAUDE_PLUGIN_ROOT/skills/orchestration/references/delegation-template.md" \
  "$CLAUDE_PLUGIN_ROOT/skills/orchestration/references/report-format.md" \
  "$CLAUDE_PLUGIN_ROOT/skills/agents/copilot-cli/SKILL.md" \
  "$CLAUDE_PLUGIN_ROOT/skills/agents/claude-cli/SKILL.md"; do
  [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
done
```

- ✅ All 9 files present
- ❌ Any missing → tell the user which files are missing and suggest reinstalling: `/plugin install cortexlink@agent-plugins`

---

## Check 3 — Hook Health

**3a. Executable bit:**

```bash
ls -la "$CLAUDE_PLUGIN_ROOT/hooks/session-start"
```

If `session-start` is not executable (no `x` in the permission bits):
- Tell the user: "session-start is not executable — this will prevent the hook from firing."
- Ask: "Should I fix the permissions now? (`chmod +x`)"
- If yes: run `chmod +x "$CLAUDE_PLUGIN_ROOT/hooks/session-start"` and confirm ✅ Fixed

**3b. jq dependency:**

```bash
jq --version 2>/dev/null || echo "NOT_FOUND"
```

- ✅ jq installed (show version)
- ❌ Not found → tell the user: the hook script requires `jq` to build its JSON output. Install with:
  - macOS: `brew install jq`
  - Linux: `sudo apt install jq`
  - Windows: `winget install jqlang.jq`

**3c. Hook test run:**

Tell the user: "Running the session-start hook to verify it produces valid output…"

```bash
"$CLAUDE_PLUGIN_ROOT/hooks/run-hook.cmd" session-start 2>&1
```

- ✅ Output is valid JSON containing `hookSpecificOutput` (Claude Code) or `additionalContext` (Copilot CLI) — show the first 200 characters of the context value as a preview
- ❌ Output is invalid JSON, empty, or contains an error — show the raw output and tell the user this means the SessionStart hook is not injecting orchestration context into sessions

---

## Check 4 — CLI Agents

**4a. Copilot CLI — installed?**

```bash
which copilot && copilot --version 2>/dev/null || echo "NOT_FOUND"
```

If found, ask the user: "Copilot CLI is installed. Do you want me to run an authentication check? (This makes a live network call.)"

If yes:

> Run `copilot models` first to get the current fastest/cheapest model ID, then substitute below.

```bash
cd "$(git rev-parse --show-toplevel 2>/dev/null || echo "$HOME")" && \
  copilot -p "ping" --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=<fastest-available> 2>/dev/null \
  && echo "AUTH_OK" || echo "AUTH_FAIL"
```

- ✅ Authenticated
- ⚠️ Auth failed → run `copilot login`
- ❌ Not installed → `brew install copilot-cli` (or see README)

**4b. Claude CLI — installed?**

```bash
which claude && claude --version 2>/dev/null || echo "NOT_FOUND"
```

If found, ask the user: "Claude CLI is installed. Do you want me to run an authentication check? (This makes a live network call.)"

If yes:

```bash
claude -p "ping" --output-format text --allowedTools "Read" --max-turns 1 --no-session-persistence 2>/dev/null \
  && echo "AUTH_OK" || echo "AUTH_FAIL"
```

- ✅ Authenticated
- ⚠️ Auth failed → run `claude auth login`
- ❌ Not installed → `npm install -g @anthropic-ai/claude-code`

---

## Check 5 — Setup State

Read `~/.claude/CLAUDE.md`. Check for a `## CortexLink` section.

- ✅ Present
- ⚠️ Absent — not required if the hook is active (Claude Code), but can be added by running `/cortexlink:setup`

Read `~/.copilot/copilot-instructions.md`. Check for a `## CortexLink` section.

- ✅ Present
- ❌ Absent — Copilot CLI needs this to have CortexLink context. Run `/cortexlink:setup` to add it.

---

## Summary

After all targeted checks complete, print this block filled in with results:

```
Platform:     [Claude Code / Copilot CLI / Unknown]

Hook
  [✅/❌]         session-start script present
  [✅/❌/✅ Fixed] session-start executable
  [✅/❌]         jq installed
  [✅/❌]         hook test run valid

Plugin files
  [✅ All 9 present / ❌ Missing: list them]

Agents
  [✅/⚠️/❌] Copilot CLI  [version / not installed / auth not checked]
  [✅/⚠️/❌] Claude CLI   [version / not installed / auth not checked]

Setup
  [✅/⚠️] ~/.claude/CLAUDE.md
  [✅/❌] ~/.copilot/copilot-instructions.md

Status: [✅ Healthy / ⚠️ Needs attention]
```

If status is healthy:
> ✅ CortexLink is healthy. The SessionStart hook will inject orchestration context automatically on every new session.

If there are unresolved issues, list each one with its fix command.
