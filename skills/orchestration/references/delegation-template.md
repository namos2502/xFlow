# Delegation Prompt Template

Use this template for every cross-CLI delegation. Keep prompts lean — no project dumps.

```
[Task]: <one sentence — what needs to be done>
[Context]: <only what the agent strictly needs>
[Success criteria]: <how the agent knows it's done>
[Report format]: Return ONLY this when finished. Plain text labels only — no **bold**, no # headers:
  STATUS: ✅ Verified / ⚠️ Partial / ❌ Failed
  SUMMARY: <1-2 sentences>
  STEPS: <bullets>
  FILES: <changed files or "none">
  ISSUES: <notes for the control center or "none">
  Max 150 words.
```

**For Complex tasks only**, append this block before the report format:

```
[Before executing]: This task is complex. In your first response, list any questions
or ambiguities — 1 turn only. Do not perform any actions until I confirm.
```

The agent surfaces questions, you refine the spec if needed, then issue the execution follow-up.

⛔ **Critical (Copilot CLI only):** Always include this line in the prompt body:
*"Return ONLY the structured report. No reasoning steps, no 'Let me...' output before the report."*

---

# Cross-Agent Chaining

Fan out to multiple Agents in parallel when tasks are independent. Pass a report excerpt (SUMMARY + STEPS) as context into the next delegation prompt — never raw output. Agents do not chain to each other; all coordination happens at the control center.

**Avoid duplicate fetches:** When multiple agents need the same source data (e.g. a PR diff, a file's contents), fetch it once in the control center and pass it as `[Context]` in each delegation prompt. Do not let each agent re-fetch the same data independently.
