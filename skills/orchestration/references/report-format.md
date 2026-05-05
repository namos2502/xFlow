# Structured Report Format

Every agent, every tier, every time:

```
STATUS: ✅ Verified / ⚠️ Partial / ❌ Failed
SUMMARY: <1-2 sentences>
STEPS:
  - <step 1>
  - <step 2>
FILES: <list or "none">
ISSUES: <notes or "none">
```

**Never return raw output or tool call dumps to the control center.**

---

# Self-Verify Before Reporting (when YOU are the agent)

Execute → Verify → (fix if needed) → Report.

1. Run the task
2. Check your own work — tests, file contents, expected state
3. If verification fails → fix, then verify again
4. Only then → write the structured report

| Status | Meaning |
|--------|---------|
| ✅ Verified | You actively checked. It works. |
| ⚠️ Partial | Some parts verified, some could not be checked. Explain in ISSUES. |
| ❌ Failed | Describe what was attempted and what failed. |

**Never mark ✅ without actually checking.**

**Status semantics for analysis tasks:**
- ✅ Verified = you read the code and completed the analysis — even if you cannot *run* it
- ⚠️ Partial = you could not access required files, or the diff was truncated
- ❌ Failed = the command failed or the data was inaccessible

⛔ Do NOT use ⚠️ just because you cannot execute the code being reviewed. Completed static analysis = ✅.

---

# Agent Detection

Run these before delegating to verify agents are available:

> Run `copilot models` to get the current fastest/cheapest Copilot model ID and substitute `<fastest-available>` below.

```bash
# Copilot CLI
which copilot 2>/dev/null && cd $(git rev-parse --show-toplevel) && \
  copilot -p "ping" --no-ask-user --no-auto-update --no-color --allow-tool='read' --model=<fastest-available> 2>/dev/null | head -1

# Claude CLI
which claude 2>/dev/null && claude -p "ping" --output-format text --allowedTools "Read" --max-turns 1 --no-session-persistence 2>/dev/null | head -1
```

If unavailable: fall back to another agent, handle natively, or tell the user to run `/cortexlink:setup`.

---

# Error Handling

| STATUS | Control center action |
|--------|-----------------------|
| ✅ | Accept, proceed |
| ⚠️ | Read ISSUES, accept partial or re-assign unverified parts |
| ❌ | Read ISSUES. Re-assign, adjust scope, or handle natively |
| No output | Retry once. If still failing, handle natively. |
| Auth failure | Tell user: `/cortexlink:setup` |

**After 2 consecutive ❌ on the same subtask — stop and discuss with the user before retrying.**
