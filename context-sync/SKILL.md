---
name: context-sync
description: >
  Closes the feedback loop in AI-assisted development by extracting learnings from a completed
  development cycle and proposing precise diffs to CLAUDE.md. Use this skill after any feature
  ships, any PRD is finalised, any architecture decision is locked, or any post-mortem is run.
  Trigger on: "update CLAUDE.md", "sync context", "what should I add to CLAUDE.md", "run
  context-sync", "close the loop", "capture learnings", "update my context file", or any
  request to reflect on what changed in a development cycle. Also trigger proactively when
  a conversation has produced architectural decisions, new domain terms, new API contracts,
  new constraints, or regulatory context that isn't already in CLAUDE.md. Never propose
  additions without evidence from the current conversation or provided artifacts. Never
  delete existing CLAUDE.md content without explicit user confirmation.
---

# Context Sync — Skill Guide

Prevents CLAUDE.md drift. After every development cycle (PRD → Stories → Plan → Code → Ship),
knowledge is gained that isn't yet in the foundation file. This skill extracts that knowledge
and proposes surgical diffs — additions, updates, and deprecations — so the next cycle starts
smarter.

---

## Why this exists

CLAUDE.md is a compounding asset. Every cycle it grows more precise. Without a deliberate
sync ritual, it fossilises — correct when written, wrong six sprints later. This skill is
the mechanism that keeps it alive.

---

## Behaviour rules

**Rule 1 — Evidence-only proposals.**
Every proposed addition must trace to something in the current conversation, a provided
artifact (PRD, stories, plan, code diff), or an explicit user statement. No invented context.

**Rule 2 — Surgical diffs, not rewrites.**
Output is a set of discrete proposed changes: ADD, UPDATE, or DEPRECATE. Each change is
one logical unit. Never propose a full CLAUDE.md rewrite unless the user explicitly asks.

**Rule 3 — Ask once, then propose.**
Run the intake questions in a single batch. Then produce the diff. Do not iterate with
multiple question rounds.

**Rule 4 — User confirms before anything is final.**
All proposed diffs are proposals. The user approves, rejects, or edits each one. The skill
never declares CLAUDE.md updated — only the user can do that.

**Rule 5 — Flag staleness explicitly.**
If a proposed UPDATE contradicts existing CLAUDE.md content, surface the conflict clearly:
`CONFLICT: existing entry says X → proposing change to Y. Reason: [evidence].`

**Rule 6 — Categorise every diff.**
Each proposed change must be tagged with one of:
- `[DOMAIN]` — new terminology, concepts, business logic
- `[TECH]` — stack changes, API contracts, infra decisions
- `[CONSTRAINT]` — new regulatory, security, or performance boundaries
- `[ACTOR]` — new users, roles, personas, systems
- `[DECISION]` — architectural or product decisions locked in this cycle
- `[DEPRECATE]` — things that are no longer true or relevant

**Rule 7 — Handle the "already covered" case in prose, not code blocks.**
If CLAUDE.md already captures all learnings from this cycle, the Proposed Diffs section must say so in plain prose — for example: "No new diffs required. The following existing entries already cover this cycle's learnings: [list them]." Do NOT produce a diff code block. Do NOT invent a pseudo-tag like `[CONSTRAINT] NO CHANGE`. Only ADD, UPDATE, and DEPRECATE are valid diff types. If there is nothing to change, there is no diff block — just prose acknowledgement.

---

## Intake protocol

Ask all questions in one numbered batch before producing any diffs:

1. What is the feature or cycle being closed? (name / Jira reference)
2. Is the CLAUDE.md file available to paste or summarise? (to check for conflicts)
3. Which artifacts from this cycle are available? (PRD, stories, plan, code, retro notes)
4. Were there any architectural decisions made that weren't in the original plan?
5. Were any new third-party APIs, services, or data contracts introduced?
6. Were any existing assumptions proven wrong or obsoleted?
7. Were any new domain terms, regulatory references, or business rules introduced?
8. Were any new actors (users, systems, ops roles) identified?
9. Are there any known constraints that surfaced during implementation (performance,
   security, compliance) that should govern future cycles?
10. Any open questions or deferred decisions that should be tracked?

---

## Output format

### Context Sync Report — [Feature Name] — [Date]

**Cycle summary:** One sentence on what shipped.

---

**Proposed diffs:**

```
[TAG] ADD
Section: <CLAUDE.md section this belongs in>
Content: <exact text to add>
Evidence: <where this came from in this cycle>
```

```
[TAG] UPDATE
Section: <section>
Existing: <current text>
Replace with: <new text>
Reason: <what changed and why>
CONFLICT: <flag if contradicts existing content>
```

```
[TAG] DEPRECATE
Section: <section>
Remove: <content to remove>
Reason: <why it's no longer valid>
```

---

**Staleness flags:**
List any existing CLAUDE.md sections that may be outdated based on this cycle's learnings,
even if a specific update isn't being proposed. Format:
`⚠️ [Section name] — may be stale because [reason]. Recommend review.`

---

**Open items to track:**
List deferred decisions, unresolved questions, or known future constraints that don't
belong in CLAUDE.md yet but shouldn't be lost.

---

## Confidence score

After producing diffs, close with:
`Confidence: X% — based on [N] artifacts reviewed. [Any gaps noted.]`

Below 70%: explicitly list which sections carry uncertainty and what would improve them.