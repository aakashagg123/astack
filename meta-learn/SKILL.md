---
name: meta-learn
description: "Skill meta-learning loop. Scans all session JSONLs, extracts correction events after Skill invocations, ranks skills by friction rate, synthesizes test inputs from real failures, and dispatches headless autoresearch pipelines. Use when: which skills are broken, skill portfolio audit, meta-learning, batch autoresearch, optimize all skills, skill friction report, which skill is causing the most wrong-approach events."
---

# Skill Meta-Learning Loop

Autoresearch optimizes one skill at a time — you pick which skill, write the test inputs, write the evals. This skill operates one level up: it reads every session you've had, identifies which skills generated the most correction events, synthesizes test inputs from real failures, and dispatches headless autoresearch runs for the candidates. The loop closes itself.

---

## Core job

```
SESSION DATA → friction extraction → skill attribution → ranking
     → hypothesis synthesis → headless autoresearch dispatch → report
```

All signal lives in `~/.claude/projects/*/**.jsonl`. All output goes to `~/.claude/signal/`.

---

## Invocation modes

**Full pipeline (default):**
`/meta-learn` — all 5 phases. Scans sessions, ranks skills, synthesizes hypotheses, dispatches autoresearch, renders report.

**Audit only (fast, no evals):**
`/meta-learn scan` — phases 1–2 only. Answers "which of my skills is most broken?" in under a minute.

**Single skill (targeted):**
`/meta-learn skill <name>` — skips phases 1–2. Reads existing friction data for one skill and runs phases 3–4 for it only.

**Report only (after overnight run):**
`/meta-learn report` — reads existing signal files and renders the HTML report. No new scans or evals.

**Headless overnight batch:**
```bash
claude -p "Run /meta-learn. Scan all sessions, rank top 3 friction skills, run autoresearch for each. Output report to ~/.claude/signal/meta-learn-$(date +%Y%m%d).html." \
  --allowedTools "Read,Write,Edit,Bash,Glob,Grep" \
  --output-format json \
  --max-turns 100
```

---

## Phase 1: Signal extraction

Scan every `~/.claude/projects/**/*.jsonl` (skip `subagents/` subdirectories — they're agent sub-sessions, not user-facing corrections).

For each session file, process events in order:

1. When you see an `assistant` event with a `Skill` tool call (`content[].type === "tool_use" && content[].name === "Skill"`), record:
   - `skill_name` = `tool_input.skill`
   - `skill_args` = `tool_input.args` (first 300 chars)
   - `event_index` = position in session

2. For each recorded Skill invocation, scan the next 8 user events for correction signals. Correction patterns (case-insensitive, partial match):
   - Direct stops: `"stop"`, `"no not that"`, `"wrong approach"`, `"don't do"`, `"never mind"`
   - Redirections: `"actually,"`, `"wait,"`, `"hold on"`, `"that's not"`, `"not what I"`
   - Failure signals: `"that's wrong"`, `"incorrect"`, `"bad approach"`, `"undo"`, `"revert this"`
   - Interruption pattern: short user message (< 30 chars) sent within 2 events of a multi-tool assistant response

3. If a correction is detected within the 8-event window after a Skill call: emit a friction event.

Write to `~/.claude/signal/events.jsonl` (append, don't overwrite — incremental scans accumulate):
```json
{"session_id": "...", "project": "...", "timestamp": "...", "skill_name": "autoresearch", "correction_text": "stop, that's the wrong...", "confidence": "high"}
```

`confidence: "high"` when correction is within 3 events of the Skill call. `"low"` for 4–8 events.

Track which session files you've already processed via `~/.claude/signal/scan-state.json` (`{"scanned": ["path1", ...], "last_run": "..."}`). Skip already-scanned files on incremental runs.

---

## Phase 2: Aggregation

Read all events from `~/.claude/signal/events.jsonl`.

For each unique `skill_name`, compute:
- `friction_count`: total high+low confidence corrections
- `high_confidence_count`: corrections within 3 events of skill call
- `session_count`: number of distinct sessions where the skill was invoked
- `friction_rate`: high_confidence_count / session_count
- `last_seen`: most recent session timestamp
- `example_corrections`: up to 5 real correction texts (prefer high-confidence, most recent)
- `skill_path`: infer from `~/.claude/skills/<name>/SKILL.md` — verify it exists

Write `~/.claude/signal/skill-friction.json`:
```json
{
  "generated_at": "2026-05-06T...",
  "sessions_scanned": 119,
  "total_friction_events": 47,
  "skills": [
    {
      "skill_name": "agent-team",
      "skill_path": "/Users/.../.claude/skills/agent-team/SKILL.md",
      "friction_count": 14,
      "high_confidence_count": 9,
      "session_count": 26,
      "friction_rate": 0.35,
      "last_seen": "2026-05-01",
      "example_corrections": ["stop, that's the wrong phase order...", "no not that — skip the planning step..."]
    }
  ]
}
```

Sort descending by `high_confidence_count`. Present the ranked table to the user before proceeding to phase 3. Pause for confirmation if running interactively (`/meta-learn` not headless).

---

## Phase 3: Hypothesis synthesis

For each top-N candidate (default N=3, configurable):

1. Read the full `SKILL.md` for the skill.
2. Read its `example_corrections` from the friction data.
3. Produce a hypothesis document at `~/.claude/signal/hypotheses/<skill-name>.json`:

```json
{
  "skill_name": "agent-team",
  "hypothesis": "One-sentence description of what's breaking and why",
  "test_inputs": [
    "User prompt that would reproduce the failure pattern...",
    "Another prompt covering a different failure instance...",
    "..."
  ],
  "evals": [
    {
      "name": "Correct phase ordering",
      "question": "Does the output proceed through phases in the documented order?",
      "pass": "Phases appear in correct sequence with no skipped steps",
      "fail": "Any phase is skipped, out of order, or collapsed with another"
    }
  ],
  "source_corrections": ["verbatim correction 1...", "verbatim correction 2..."]
}
```

Rules for test inputs:
- Derive from real correction examples — don't invent scenarios not reflected in actual failure data
- Cover at least 3 distinct failure modes if enough corrections exist
- Keep each prompt under 200 chars (autoresearch runs them many times)

Rules for evals:
- Binary only — yes/no, no scales
- 3–6 evals per skill (more creates overfitting to the eval set)
- At least one eval must directly address the correction pattern (not general quality)

---

## Phase 4: Headless autoresearch dispatch

For each synthesized hypothesis, construct and execute a `claude -p` invocation:

```bash
claude -p "Run /autoresearch on the skill at <skill_path>.

Test inputs to use (run each input at least once per experiment cycle):
<JSON array of test inputs from hypothesis file>

Eval criteria (binary yes/no):
<JSON array of evals from hypothesis file>

Runs per experiment: 5. Budget cap: 15 experiments.
Output results to ~/.claude/signal/autoresearch-<skill_name>-$(date +%Y%m%d).json.
Name the optimized version: <skill_name>-meta-optimized." \
  --allowedTools "Read,Write,Edit,Bash,Glob,Grep" \
  --output-format json \
  --max-turns 60
```

Run sequentially — one skill at a time. Parallel runs saturate tokens and produce degraded outputs.

Log start time, end time, and exit code for each run in `~/.claude/signal/dispatch-log.json`.

---

## Phase 5: Report

Read all autoresearch result files from `~/.claude/signal/autoresearch-*.json`. Cross-reference with `skill-friction.json` for before/after context.

Generate `~/.claude/signal/meta-learn-report-<YYYYMMDD>.html` — a single self-contained HTML file with inline CSS and Chart.js from CDN:

**Dashboard sections:**
1. **Skills ranked by friction** — bar chart (skill name on Y, friction count on X, colored by confidence level)
2. **Autoresearch results table** — skill | baseline score | final score | improvement % | experiments run | top change
3. **Remaining failure patterns** — what each skill still gets wrong post-optimization (from autoresearch changelogs)
4. **Optimized files available** — paths to the `<skill>-meta-optimized.md` files with a note: "Review before applying — original SKILL.md unchanged"

Open the report automatically: `open ~/.claude/signal/meta-learn-report-<date>.html`

---

## Output files

```
~/.claude/signal/
├── scan-state.json                    # which sessions have been scanned
├── events.jsonl                       # all friction events (append-only)
├── skill-friction.json                # aggregated ranking (overwritten each run)
├── hypotheses/
│   ├── agent-team.json
│   ├── autoresearch.json
│   └── ...
├── autoresearch-agent-team-20260506.json    # headless autoresearch output
├── autoresearch-autoresearch-20260506.json
├── dispatch-log.json                  # run timing + exit codes
└── meta-learn-report-20260506.html    # final dashboard
```

---

## Failure handling

**No Skill calls found in sessions:** Report "0 friction events attributed to named skills. Sessions may predate Skill tool logging or skills were invoked without the Skill tool." Show unattributed correction events count so the user knows signal exists but attribution failed.

**Skill path not found:** Skip that skill in phases 3–4. Log it as "skill not on disk — may have been renamed or deleted."

**Headless autoresearch exits non-zero:** Log it in dispatch-log.json. Continue with remaining skills. Include a "run failed" row in the final report — don't silently skip it.

**events.jsonl is empty on first run:** This is normal — scan-state.json didn't exist yet. On first run, scan all sessions from scratch (no state file = no sessions marked scanned).

---

## Design constraints

- Never mutate any `SKILL.md` directly — autoresearch handles that with its own non-destructive copy pattern
- The original SKILL.md for each skill is always preserved
- All writes go to `~/.claude/signal/` — never to skill directories during the meta loop itself
- `events.jsonl` is append-only — incrementally accumulates signal across runs
- Running `/meta-learn scan` twice on the same session set should produce identical output (idempotent via scan-state.json)
