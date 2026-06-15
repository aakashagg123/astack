---
name: marketing-brief
description: >
  Create a rock-solid, implementation-ready marketing brief from a growth strategy, product analytics, and the features that have actually shipped. Use this skill when a product manager or product marketer needs to turn a strategy map or a raw campaign idea into a brief that designers, copywriters, and automation builders can execute from without guessing. Trigger on: write a marketing brief, draft a campaign brief, brief this campaign, create a GTM brief, turn this strategy into a brief, review my brief, what's missing in this brief, brief for a launch / lifecycle / re-engagement. The brief is the contract — every downstream asset and automation traces back to it. Pairs with /growth-discovery upstream (which produces the Strategy Map) and /marketing-automation-builder downstream (which builds the workflow). Never generate the brief before completing the upfront Q&A. Always ask clarifying questions in a single batch.
---

# Marketing Brief — Skill Guide

Produces a single, complete marketing brief grounded in three sources of truth: the growth strategy, the product analytics, and the features that have actually shipped. A brief that invents an audience, promises a feature that isn't live, or sets an unmeasurable goal is not a brief — it's a wish. Works in two modes:

- **Generate mode**: User has a Strategy Map (from `/growth-discovery`) or a raw idea. Skill runs Q&A, then produces a full brief saved to `/briefs`.
- **Review mode**: User has an existing brief. Skill audits it against the standard, flags gaps, and offers to fill them.

---

## Behaviour rules

**Rule 1 — Ask first, write never.** Never generate brief content before the upfront Q&A. All clarifying questions go in a single numbered batch. No silent assumptions about audience, channel, or what's shipped.

**Rule 2 — Rooted in what shipped, not what's planned.** Every claim, hook, and value proposition must map to a feature that is *live* or a benefit that is *real today*. Mark any roadmap-dependent message explicitly: `[ Gated on feature X — do not promise until live ]`. Marketing ahead of the product is the fastest way to burn trust.

**Rule 3 — Analytics-grounded, not assertion-grounded.** Audience size, funnel baselines, and segment behaviour come from product analytics with a named source and date. If unavailable: `[ Owner to provide from analytics ]`. No invented numbers.

**Rule 4 — Every KPI is measurable.** Each objective has a KPI in `Given/When/Then` form with a baseline, a target, a date, and a measurement source. A KPI you can't instrument is not a KPI.

**Rule 5 — Sentence case throughout.** All content in sentence case. No ALL-CAPS body. No trailing full stops on labels.

**Rule 6 — One message per row.** In the messaging matrix, never bundle multiple value propositions, audiences, or channels into one row. Split them.

**Rule 7 — File output is mandatory.** Save to `/briefs/{slug}.md` in the working directory. Create `/briefs` if absent. Confirm the path.

**Rule 8 — Confidence score at end.** Close with a 0–100% score on information completeness. Below 80%: name the soft sections.

---

## Upfront clarification protocol

If a Strategy Map from `/growth-discovery` is provided, read it first and only ask what it doesn't already answer. Otherwise fire all questions in **one numbered batch**.

**Strategy and objective**
1. What is the growth motion and the single objective this brief serves? (acquisition / activation / retention / monetisation / re-engagement / launch)
2. What is the measurable target — baseline, goal, date, measurement source?
3. What is the cost of doing nothing — is the metric decaying or stable?

**Audience (from analytics)**
4. Who is the primary segment? Size, defining behaviour, and the analytics source that defines it.
5. What is the reachable count after consent / opt-in filtering, by channel?
6. What does this segment already do in the product — the behavioural baseline?

**Product truth**
7. Which *shipped* features or benefits anchor the message? Confirm each is live.
8. What is explicitly off-limits to promise (unreleased, beta, deprecated)?

**Message and offer**
9. What is the single most important thing the audience must understand or do?
10. Is there an offer, incentive, or hook? What are its terms and limits?
11. What is the brand voice and any non-negotiable wording or claims constraints?

**Channel and execution**
12. Which channels carry this — web, in-app, email, push, WhatsApp, paid? Which is primary?
13. What are the frequency caps, quiet hours, and consent/regulatory limits?
14. What is the timeline — launch date, duration, key milestones?
15. Who are the reviewers and the final approver?

---

## Output structure

Generate in this exact order. Save to `/briefs/{slug}.md`.

### Header block

```
Brief: [name]
Version: 1.0
Status: Draft
Motion: [acquisition | activation | retention | monetisation | re-engagement | launch]
Owner: [marketer — default to user]
Date: [today]
Linked to: [Strategy Map / campaign ticket / "[ To be linked ]"]
```

### Reviewer matrix

| Domain | Reviewer | Status |
|---|---|---|
| Product marketing | [Name] | Pending |
| Brand / content | [Name] | Pending |
| Analytics | [Name] | Pending |

### Executive summary

One paragraph a non-marketer can read: what campaign, for whom, why now, expected outcome.

### Objective and KPIs

**Objective:** one sentence — the single thing this campaign moves.

**KPI table:**

| KPI | Baseline | Target | Date | Measurement source |
|---|---|---|---|---|
| [primary — lagging] | | | | |
| [leading indicator] | | | | |
| [guardrail metric — e.g. unsubscribe / complaint rate] | | | | |

Always include at least one **guardrail metric** — the thing that must *not* get worse (opt-out rate, complaint rate, CAC).

### Audience

| Segment | Defining behaviour | Size (base) | Reachable (by channel) | Consent status | Source |
|---|---|---|---|---|---|

One row per segment. Reachable count, not base, is the real audience.

### Product truth — what's shipped

| Message anchor | Backing feature / benefit | Live? | Evidence |
|---|---|---|---|

Every anchor maps to something real today. Roadmap items get `[ Gated — do not promise ]`.

### Messaging matrix

| Audience | Insight / pain | Value proposition | Proof point | Primary CTA |
|---|---|---|---|---|

One message per row. Never bundle. The value proposition ties to a shipped feature from the section above.

### Offer and incentive

Terms, eligibility, value, expiry, and any abuse/cap controls. If none: "No offer — value-led message only."

### Channel plan

| Channel | Role | Audience slice | Frequency cap | Constraint |
|---|---|---|---|---|
| [e.g. WhatsApp] | Primary nudge | [opt-in slice] | [1 / week] | [24h session window, template approval] |
| [e.g. In-app] | | | | |

State which channel is primary and the sequencing logic across channels.

### Decisions register

| Decision | Rationale | Made by | Date |
|---|---|---|---|

Capture every choice that, if reversed, changes the campaign — channel priority, audience cut, offer terms.

### Assumption register

| Assumption | If false, what breaks | Validate by |
|---|---|---|

Make implicit assumptions explicit — "assumes WhatsApp templates clear approval before launch", "assumes the 18% baseline holds at scale".

### Constraints and non-scope

Frequency caps, quiet hours, consent/regulatory limits (DND, opt-in, data residency), brand rules, and what this campaign is explicitly *not* doing.

### Measurement plan

How each KPI is instrumented: events, funnels, attribution window, holdout/control group, and the read date. State the holdout explicitly — either name the control group and its size, or write "no holdout — results are directional, not causal." A brief that leaves holdout unstated cannot prove a lift; the declaration is mandatory, not optional.

### Open questions

| Question | Answer / decision | Owner | Target date |
|---|---|---|---|

---

## HTML companion (mandatory)

After saving the markdown, generate a standalone, print-ready HTML brief at `/briefs/{slug}.html`. This is a designed document, not a markdown dump. Use a clean, professional, Anthropic-style design system — **never any client-specific or proprietary tokens**.

Delegate styling to `/brand-guidelines` (Anthropic colours and typography) and structural rendering to `/web-artifacts-builder` when available. If building inline, use this token base in `<head>`:

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap">
<style>
:root {
  --ink-900:#181818; --ink-700:#3a3a37; --ink-500:#6b6b66; --ink-200:#e4e2dd;
  --paper:#ffffff; --paper-warm:#f7f5f0; --paper-subtle:#efece6;
  --accent-600:#c2410c; --accent-100:#ffedd5;   /* Anthropic clay accent */
  --success-600:#15803d; --success-100:#dcfce7;
  --warning-600:#b45309; --warning-100:#fef3c7;
  --error-600:#b91c1c;   --error-100:#fee2e2;
  --font-body:'Inter',system-ui,sans-serif; --font-mono:'JetBrains Mono',monospace;
}
* { box-sizing:border-box; margin:0; padding:0; }
body { font-family:var(--font-body); background:var(--paper-warm); color:var(--ink-700); font-size:14px; line-height:1.6; }
</style>
```

Render each markdown section as a card. Map KPI / guardrail / assumption tables to styled tables; flag `[ Gated ]` anchors and unanswered open questions with a warning badge. Apply the same UX content rules below to rendered text.

### UX content rules for HTML output

1. Sentence case on all headings, labels, table headers.
2. No trailing full stops on labels, headings, table cells.
3. Dates `DD MMM YYYY`; no ISO 8601 in display.
4. Colons: no space before, one space after — `Status: Active`.
5. Alternatives: whitespace around `/` — `web / app`.
6. No underlined text. No raw hex — all colours via `var(--token)`.
7. Monospace for IDs, KPI codes, version numbers.

### HTML quality check

1. Inter + JetBrains Mono + full token block in `<head>`.
2. Every markdown section rendered as a card. No unstyled tables.
3. No raw hex in any style. No client-specific or proprietary token names anywhere.
4. Gated anchors and open questions visibly flagged.
5. Opens in a browser fully designed; print-friendly.

---

## Review mode

Audit a provided brief against each section:

| Section | Status | Gap / issue |
|---|---|---|
| Header | ✅ / ⚠️ / ❌ | |
| Objective and KPIs | ✅ / ⚠️ / ❌ | |
| Audience (analytics-grounded) | ✅ / ⚠️ / ❌ | |
| Product truth (shipped) | ✅ / ⚠️ / ❌ | |
| Messaging matrix | ✅ / ⚠️ / ❌ | |
| Offer | ✅ / ⚠️ / ❌ | |
| Channel plan | ✅ / ⚠️ / ❌ | |
| Decisions register | ✅ / ⚠️ / ❌ | |
| Assumption register | ✅ / ⚠️ / ❌ | |
| Measurement plan (holdout) | ✅ / ⚠️ / ❌ | |

Legend: ✅ Complete | ⚠️ Partial | ❌ Missing. After the report, offer to fill gaps. Save back, bump version.

---

## Anti-patterns

- **Promising what hasn't shipped.** The single fastest trust-killer. Every anchor maps to live product truth.
- **Assertion-grounded audiences.** "Our power users" with no analytics definition is a guess. Cite the source and the count.
- **KPI without a baseline or a holdout.** You can't prove a lift you never measured a before-value for, and you can't prove causation without a control.
- **No guardrail metric.** A campaign that hits the goal while doubling unsubscribes is a loss. Always track what must not get worse.
- **Bundled messages.** One row, one value proposition, one audience, one CTA.
- **Ignoring consent reality.** Reachable count ≠ base. Brief to the slice you can legally and consensually reach.

---

## Confidence score

```
Confidence score: [0–100]%
Basis: [one sentence — what's solid vs. soft]
```

Below 80%: name the sections carrying low confidence and why.
