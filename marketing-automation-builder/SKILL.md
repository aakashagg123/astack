---
name: marketing-automation-builder
description: >
  Turn an approved marketing brief into a complete marketing automation workflow — the trigger / branch / wait / message / exit logic of a customer journey — rendered as a beautified, standalone HTML artifact with marketing KPIs wired in. Use this skill when a product manager or product marketer has a brief (ideally from /marketing-brief) and needs the executable journey: lifecycle flows, onboarding sequences, re-engagement journeys, drip campaigns, abandoned-cart or activation nudges across web, app, email, push, and WhatsApp. Trigger on: build the automation, design the journey, create the workflow, map the drip / lifecycle / nudge flow, turn this brief into a journey, build the campaign flow, automation diagram, journey builder. Output is a self-contained HTML file that renders the journey visually with message previews, branch logic, and a KPI dashboard — using a clean Anthropic-style design system. Never use client-specific or proprietary design tokens. Always build from an approved brief; if none exists, route to /marketing-brief first.
---

# Marketing Automation Builder

You convert an approved marketing brief into an executable customer journey and render it as a beautiful, standalone HTML artifact. The journey is the *how*; the brief is the *what*. Every node in the journey traces back to a brief objective, and every objective shows up as a KPI in the dashboard.

This is the build step of the growth pipeline — the marketing twin of `/web-artifacts-builder` (the artifact) fused with `/tech-development-plan` (the sequenced logic).

---

## Behaviour rules

**Rule 1 — Build from a brief, never from a vibe.** If no approved brief is provided, route to `/marketing-brief` first. The journey inherits its audience, KPIs, channels, and constraints from the brief — do not invent them.

**Rule 2 — Every node traces to the brief.** Each message, branch, and exit maps to a brief objective or constraint. A node that serves no brief objective gets cut.

**Rule 3 — KPIs are wired in, not bolted on.** The HTML includes a KPI dashboard built from the brief's KPI table — baseline, target, and the measurement point in the journey where each is read. A journey you can't measure is a guess.

**Rule 4 — Honour the constraints.** Frequency caps, quiet hours, consent / opt-in gates, and channel session windows (e.g. WhatsApp 24h) are encoded as explicit nodes or guards in the journey. Constraints from the brief are non-negotiable.

**Rule 5 — Anthropic design system only.** The HTML uses a clean, warm, Anthropic-style design system. **Never use any client-specific or proprietary tokens, names, or palettes anywhere — not in tokens, not in comments, not in copy.** Delegate styling to `/brand-guidelines` and complex rendering to `/web-artifacts-builder` where available.

**Rule 6 — Sentence case. No trailing full stops on labels. Confidence score at end.**

---

## Phase 1 — Read the brief, model the journey

**Step 0 — confirm an approved brief exists.** Before anything else, verify you have an approved marketing brief (ideally from `/marketing-brief`). If none is provided, STOP and route to `/marketing-brief` — do not invent an audience, KPIs, or objective to proceed. The brief-gate is non-skippable.

Read the brief. Extract: objective, audience segments, channels and primary channel, KPIs, offer, constraints, and the messaging matrix. Then model the journey as a directed graph of nodes.

**Node types:**

| Node | Purpose | Carries |
|---|---|---|
| **Entry trigger** | What enrols a user | event / segment / schedule + entry cap |
| **Message** | A send | channel, template, copy, CTA, the brief message it realises |
| **Wait / delay** | Pacing | duration, quiet-hours guard |
| **Branch** | Decision split | condition (behaviour, attribute, channel state) + each path |
| **Goal / conversion** | Success node | the KPI event it fires |
| **Guard** | Constraint enforcement | frequency cap, consent check, session window |
| **Exit** | Removal from journey | converted / opted-out / completed / fatigue cap |

Model the happy path first, then every branch, then every exit. A journey with one exit is incomplete — model at least three exit types (converted, opted out, fatigue-capped, completed); users leave for many reasons, and a journey that only models conversion will trap and over-message everyone else.

---

## Phase 2 — Journey specification (markdown)

Write the journey as a spec before rendering. Save to `/journeys/{slug}.md`.

### Header

```
Journey: [name]
Version: 1.0
Built from brief: [brief slug / link]
Motion: [from brief]
Primary channel: [channel]
Date: [today]
```

### Journey map (node table)

| Node ID | Type | Channel | Trigger / condition | Action / message | Next node(s) | Brief reference |
|---|---|---|---|---|---|---|
| N1 | Entry | — | [event/segment] | Enrol | N2 | Objective |
| N2 | Message | WhatsApp | immediate | [template + CTA] | N3 (wait) | Message row 1 |
| N3 | Wait | — | 48h, skip quiet hours | — | N4 | Constraint |
| N4 | Branch | — | converted? | yes → Goal / no → N5 | — | KPI |

One row per node. Every row names the brief element it serves.

### Guards and constraints

List every frequency cap, consent gate, quiet-hour rule, and channel session window, and which nodes enforce them.

### Exit criteria

| Exit | Condition | What fires |
|---|---|---|
| Converted | goal event | KPI conversion, remove from journey |
| Opted out | opt-out event | suppress, log guardrail metric |
| Fatigue cap | N messages reached | remove, do not re-enrol for X days |
| Completed | reached terminal node | mark complete |

### KPI dashboard spec

| KPI | Baseline | Target | Measured at node | Source |
|---|---|---|---|---|

Pull directly from the brief's KPI table. Map each KPI to the journey node where it's read. Always include the guardrail metric (opt-out / complaint rate).

---

## Phase 3 — Beautified HTML artifact (mandatory)

Render the journey as a single, self-contained, standalone HTML file at `/journeys/{slug}.html`. Opening it in a browser shows: the journey flow visually, each message previewed, branch logic, guards, exits, and the KPI dashboard — no build step, no backend. Use vanilla JS for any interactivity (collapsible nodes, message previews).

**Delegate first:** if `/web-artifacts-builder` is available, use it for complex interactive rendering; use `/brand-guidelines` for the Anthropic palette and typography. If building inline, use this base:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>[Journey name] — Automation Workflow</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap">
<style>
:root {
  --ink-900:#181818; --ink-700:#3a3a37; --ink-500:#6b6b66; --ink-200:#e4e2dd;
  --paper:#ffffff; --paper-warm:#f7f5f0; --paper-subtle:#efece6;
  --accent-600:#c2410c; --accent-100:#ffedd5;        /* Anthropic clay */
  --node-msg:#1d4ed8;   --node-msg-bg:#eff6ff;
  --node-wait:#7c3aed;  --node-wait-bg:#f5f3ff;
  --node-branch:#b45309;--node-branch-bg:#fef3c7;
  --node-goal:#15803d;  --node-goal-bg:#dcfce7;
  --node-exit:#b91c1c;  --node-exit-bg:#fee2e2;
  --font-body:'Inter',system-ui,sans-serif; --font-mono:'JetBrains Mono',monospace;
}
* { box-sizing:border-box; margin:0; padding:0; }
body { font-family:var(--font-body); background:var(--paper-warm); color:var(--ink-700); font-size:14px; line-height:1.6; }
.journey { max-width:1040px; margin:0 auto; padding:32px 24px 64px; }
.journey-header { background:linear-gradient(135deg,var(--ink-900),var(--accent-600)); color:#fff; border-radius:16px; padding:36px; margin-bottom:28px; }
.kpi-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(180px,1fr)); gap:14px; margin-bottom:28px; }
.kpi-card { background:var(--paper); border:1px solid var(--ink-200); border-radius:12px; padding:18px; }
.kpi-card .num { font-size:30px; font-weight:700; color:var(--accent-600); }
.flow { display:flex; flex-direction:column; gap:0; }
.node { border-radius:12px; padding:16px 20px; border:1px solid var(--ink-200); background:var(--paper); position:relative; }
.node--msg { border-left:4px solid var(--node-msg); }
.node--wait { border-left:4px solid var(--node-wait); }
.node--branch { border-left:4px solid var(--node-branch); }
.node--goal { border-left:4px solid var(--node-goal); }
.node--exit { border-left:4px solid var(--node-exit); }
.connector { width:2px; height:24px; background:var(--ink-200); margin:0 auto; }
.msg-preview { background:var(--paper-subtle); border-radius:10px; padding:12px 14px; margin-top:10px; font-size:13px; }
.badge { display:inline-block; padding:2px 10px; border-radius:12px; font-size:11px; font-weight:600; }
@media print { body{background:#fff;} .node{break-inside:avoid;} }
@media (max-width:768px){ .journey{padding:16px 12px;} .journey-header{padding:24px;} }
</style>
</head>
```

**Render the flow** as a vertical sequence of `.node` cards connected by `.connector` lines, colour-coded by node type. Branches render as a node with two labelled outgoing paths. Each message node embeds a `.msg-preview` showing the channel, the actual copy, and the CTA — so a reviewer reads exactly what the user will receive. The KPI dashboard renders as a `.kpi-grid` of `.kpi-card`s at the top (baseline → target, with the read-point named).

### UX content rules for HTML output

1. Sentence case on all headings, node labels, buttons, table headers.
2. No trailing full stops on labels, node titles, CTAs.
3. Dates `DD MMM YYYY`; no ISO 8601 in display.
4. Colons: no space before, one space after.
5. Alternatives: whitespace around `/` — `web / app`, `push / email`.
6. No underlined text. No raw hex in any inline style — all via `var(--token)`.
7. Monospace for node IDs, template IDs, KPI codes.

### HTML quality check

1. Inter + JetBrains Mono + full token block in `<head>`.
2. Every journey node from the spec rendered as a `.node` card, correctly colour-typed.
3. Every message node shows a real copy preview with channel and CTA.
4. KPI dashboard present at top, sourced from the brief, with baseline → target and read-point.
5. Guards (frequency cap, consent, quiet hours) and all exits visibly rendered.
6. **No raw hex, no client-specific or proprietary tokens or names anywhere** — Anthropic system only.
7. Opens in a browser fully designed and interactive without a build step; print-friendly.

---

## Phase 4 — Handoff and validation

- Confirm to the user: "Journey saved to `/journeys/{slug}.md` — beautified workflow at `/journeys/{slug}.html`."
- State the trace: every node → its brief reference, so the user can verify nothing drifted from the brief.
- After launch, route performance data back through `/campaign-insights` to produce a Brief Delta, then rebuild the journey with the changes. That closes the loop.

---

## Anti-patterns

- **Building without a brief.** A journey invented from scratch has no measurable objective and no audience truth. Start from `/marketing-brief`.
- **One exit.** Real users leave for many reasons. Model converted, opted-out, fatigue-capped, and completed exits — or you'll trap and over-message people.
- **KPIs as decoration.** A dashboard not wired to actual journey read-points proves nothing. Map each KPI to the node where it's measured.
- **Ignoring guards.** Frequency caps and consent windows encoded as afterthoughts get violated at scale. Make them explicit nodes.
- **Raw markdown-to-HTML.** The artifact is a designed document with visual flow and message previews — not a dumped table.
- **Any client-specific or proprietary styling.** This stack is portable. Anthropic design system only, every time.

---

## Confidence score

```
Confidence score: [0–100]%
Basis: [one sentence — brief completeness and journey coverage]
```

Below 80%: name which nodes or KPIs rest on assumptions the brief didn't fully resolve.
