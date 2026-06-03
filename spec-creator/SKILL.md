---
name: spec-creator
description: Create a comprehensive, implementation-ready specification document for anything you want to build — a feature, system, API, tool, flow, skill, process, or product. Use this skill when you need to turn a raw idea into a structured spec that engineers, designers, or AI coding agents can execute from. Trigger on: write a spec, create a spec, spec out this feature, spec this, I want to spec, detailed specification, spec document, spec for a feature, help me spec this out, I need a spec, build a spec, create a specification, write a specification, or any request to formally document what needs to be built before building it. Always invoke this skill before coding begins on a non-trivial piece of work. The spec is always saved as a Markdown file in a /specs folder (created automatically if missing). This skill merges PRD-level business clarity with user-story-level implementation detail into one artifact. Never skip the upfront Q&A — ask all clarifying questions in a single batch before writing anything.
---

# Spec Creator — Skill Guide

Produces a single, complete specification document for anything the user wants to build. Works in two modes:

- **Generate mode**: User has a raw idea. Skill runs Q&A, then produces a full spec saved to `/specs`.
- **Review mode**: User has an existing spec or partial doc. Skill audits it against the standard, flags gaps, and offers to fill them.

---

## Behaviour rules

**Rule 1 — Ask first, write never.**
Never generate spec content before completing the upfront Q&A. All clarifying questions go in a single numbered batch. No partial output followed by late questions. No silent assumptions.

**Rule 2 — No filler, no placeholders.**
If information is missing after Q&A, mark the field explicitly:
`[ Insufficient information — owner to provide ]`

**Rule 3 — Sentence case throughout.**
All section content in sentence case. No ALL-CAPS body text. Headers are labels, not marketing copy.

**Rule 4 — File output is mandatory.**
Every generated spec is saved to `/specs/{spec-name-slug}.md` in the current working directory. Create the `/specs` directory if it does not exist. Confirm the file path to the user after saving. Do not only print the spec to chat and skip saving.

**Rule 5 — Adapt to spec category.**
A feature spec, an API spec, a data pipeline spec, and a skill spec each have different emphasis. Mark sections as "N/A — not applicable for this spec type" when genuinely irrelevant (e.g., UI/UX for a backend-only API). Never omit a section silently.

**Rule 6 — One requirement per row.**
In the requirements table, never bundle multiple actions, actors, or data objects into one row. If a requirement covers more than one discrete action, split it.

**Rule 7 — Test cases map 1:1 to scenarios.**
Every scenario defined in the acceptance criteria must have exactly one corresponding test case. No orphan test cases. No untested scenarios.

**Rule 8 — Confidence score at end.**
Close every completed spec with a confidence score (0–100%) reflecting information completeness. Below 80%: list which sections carry uncertainty and why.

**Rule 9 — HTML output is mandatory.**
Every spec gets a beautified HTML companion file saved alongside the markdown. The HTML uses a clean, professional design system (Inter font, neutral tokens) and follows standard UX content guidelines (sentence case, consistent date and currency formatting). The HTML is a standalone, print-ready document — not a raw markdown-to-HTML dump. Never skip the HTML generation step.

**Rule 10 — HTML is the UI source of truth for implementation.**
When the spec has a UI/UX requirements section, the companion HTML file is the **visual contract** — not just documentation. The HTML must include interactive wireframes that show the exact layout, component structure, states, and interaction patterns that the implementation must match. Any agent or engineer implementing UI from this spec MUST read the companion HTML before writing components. The markdown describes *what*; the HTML shows *how it looks and behaves*.

Implementation protocol:
- Before implementing any UI component described in the spec, read the companion HTML file (`/specs/{slug}.html`) and extract: layout structure, component hierarchy, state transitions, interactive behaviors (accordion, filter pills, conditional sections), and visual patterns (badges, progress bars, cards).
- The HTML wireframe's DOM structure, CSS token usage, and interaction patterns are binding. Deviations from the HTML wireframe require explicit user approval.
- When the spec's UI/UX section says "see companion HTML" or the HTML contains a `wireframe-container` section, that section is the implementation reference — not the prose description in the markdown.

---

## Upfront clarification protocol

Scan the input for gaps across all spec sections. Fire all questions in **one numbered batch** — never split across multiple messages.

**Context and purpose**
1. What is this spec for? Classify it: feature / system / API / tool / flow / skill / process / product.
2. What is the single-sentence problem this solves? What happens if it isn't built?
3. Who are the primary actors — end users, internal operators, external systems, or a mix? What roles and permissions exist?
4. What OKRs, business goals, or product bets does this tie to?

**Scope**
5. What is explicitly in-scope? (features, flows, integrations, user journeys)
6. What is explicitly out of scope? Are there linked specs or deferred items?
7. Is this net-new, or extending an existing module/system/service?

**Functional behaviour**
8. What triggers this? (user action, API call, scheduled event, system state change, webhook, etc.)
9. What is the exact expected output or end state when everything works correctly?
10. What are the known negative scenarios and edge cases the spec must handle?
11. What hard business rules or domain constraints govern this? (e.g., skip logic, status gates, thresholds, priority rules)

**Technical detail**
12. Which external systems, APIs, or internal services does this touch? Who owns them?
13. Which data objects, tables, or entities are created, read, updated, or deleted?
14. Are there non-functional requirements — performance SLAs, concurrency limits, security, data retention?
15. Is there a UI/UX component, or is this backend/system only?

**Quality and governance**
16. What are the key success metrics? (at minimum: one product KPI and one technical KPI)
17. Are failure modes and rollback provisions defined?
18. Are there monitoring, alerting, or observability requirements?
19. Are there regulatory, compliance, or internal governance constraints?
20. Who are the reviewers? (tech, product, business, compliance)
21. Target phase, milestone, or timeline?
22. Are there open questions or pending decisions that should be tracked?

Add domain-specific follow-ups where the context demands it. For AI/ML specs: add dataset, model, evaluation, and deployment questions. For fintech: add regulatory, RBI/SEBI compliance, and audit trail questions.

---

## Output structure

Generate in this exact section order. Every section header is a label. Content is prose or structured lists in sentence case. Save the completed document to `/specs/{slug}.md`.

---

### Header block

```
Spec: [name]
Version: 1.0
Status: Draft
Category: [feature | system | API | tool | flow | skill | process | product]
Author: [author — default to user if unspecified]
Date: [today's date]
Linked to: [Jira EPIC / PRD link / "[ To be linked ]"]
```

---

### Reviewer matrix

| Domain | Reviewer | Status |
|---|---|---|
| [e.g., Engineering] | [Name] | Pending |
| [e.g., Product] | [Name] | Pending |

If reviewer names are unknown: `[ Owner to assign ]`.

---

### Abbreviations and definitions

| Term | Definition |
|---|---|
| [e.g., DSCR] | [e.g., Debt Service Coverage Ratio] |

Include all domain terms, acronyms, internal system names, and regulatory abbreviations used in the spec.

---

### Executive summary

One paragraph. Readable by a non-technical stakeholder. Covers: what is being built, why it matters, and what outcome is expected.

---

### Problem statement and supporting data

- What is the problem?
- Who is affected and how?
- What is the consequence of not solving it?
- Quantitative evidence (metrics, defect rates, user research, competitive context). Reference source and date. If unavailable: `[ Owner to provide supporting data ]`.

---

### Goals, OKRs, and success criteria

**OKR table:**

| Goal | Objective | Key result |
|---|---|---|
| | | |

**Success metrics:**
List leading and lagging indicators. At minimum: one product KPI, one operational KPI, one technical KPI.

**Definition of done:**
An explicit checklist. The spec is only closeable when every item is checked. Adapt to spec type — typical items: code reviewed, unit tests passing, integration tests passing, UAT sign-off, documentation updated, feature demonstrated, logs implemented for all success and failure states.

---

### Scope

**In-scope:**
Bulleted list of features, flows, integrations, and user journeys explicitly covered.

**Out of scope:**
Bulleted list of deferred or excluded items. Link to relevant specs or tickets where applicable.

---

### Actors and permissions

| Actor | Type | Role description | Access level |
|---|---|---|---|
| [e.g., RM] | User | Initiates lead capture | Read/Write on stages 1–2 |
| [e.g., Payment gateway] | System | Processes disbursement | API trigger only |

---

### Functional requirements

| Priority | Module | User story | Detailed requirement | Tech notes | Ticket |
|---|---|---|---|---|---|
| P0 | | As a [actor], I want to [action] so that [outcome]. | [Exhaustive behaviour: edge cases, validations, business rules] | [API contracts, data changes, integration specifics] | |

Priority: P0 (must-have), P1 (should-have), P2 (nice-to-have).
One row per discrete requirement. Never bundle multiple actions or actors into one row.

**Anti-pattern — wrong (bundled):**

| P0 | Notifications | As a system... | Create, send, and log payment reminders | | |

**Correct (split):**

| P0 | Notifications | As a system, I want to create a notification record when payment is due so that the event is tracked. | [Detail] | | |
| P0 | Notifications | As a system, I want to send push/SMS/email to the user so that they are reminded before the due date. | [Detail] | | |
| P1 | Notifications | As a system, I want to log delivery success/failure so that ops can audit outcomes. | [Detail] | | |

---

### Acceptance criteria

**System story:**
One concise statement of what this spec delivers and why it exists.

**Core assumptions:**
Conditions that must hold true for this spec to be implementable. If violated, the spec breaks.

**Pre-conditions:**
System states or data that must exist before this spec can execute.

**Expected behaviour (scenarios):**
List every scenario — positive, negative, and edge cases.

- Scenario 1: [condition] → [expected output or state change]
- Scenario 2: [condition] → [expected output or state change]
- ...

Do not leave any scenario implicit. If a business rule applies, cite it inline.

**Business rules validation:**
Domain logic the system must enforce. Each rule is a declarative constraint tied to the domain model. Examples: skip conditions, status gates, threshold enforcement, priority logic.

---

### Non-functional requirements

| NFR dimension | Requirement |
|---|---|
| Performance | [p95/p99 latency targets, throughput, concurrent users] |
| Reliability | [uptime SLA, recovery time objective] |
| Security | [auth model, access control, encryption, PII handling] |
| Scalability | [expected load growth, scaling strategy] |
| Compliance | [regulatory constraints, audit trail requirements] |
| Back-of-envelope | [estimated users, file sizes, API call frequency, storage, cost] |

---

### System and data architecture

**Systems involved:**

| System | Interaction type | Method | Owner |
|---|---|---|---|
| [e.g., Supabase] | Read/Write | REST API | [Owner] |
| [e.g., CIBIL API] | Trigger/Receive | Webhook | [Owner] |

**Data model impact:**

| Table / entity | Operation | Fields affected |
|---|---|---|
| [e.g., loan_applications] | Update | [status, cam_data] |

---

### UI/UX requirements

Describe interface behaviour to meet user expectations: feedback messages, loading states, empty states, error displays, confirmation flows, accessibility requirements.

If this is a backend-only or system spec, state: "No UI/UX component — backend/system scope only."

---

### Error handling and edge cases

For every failure mode in the acceptance criteria scenarios, define:
- What the system does on failure
- How the error is logged and surfaced
- Whether a retry mechanism applies
- Rollback trigger condition and stable state to revert to

| Failure mode | System response | Logged? | Retry? | Rollback state |
|---|---|---|---|---|
| [e.g., CIBIL API timeout] | [Show error banner, preserve form state] | Yes | 3x with backoff | [Previous stage state] |

---

### Monitoring and observability

| Event | Log level | Alert trigger | Who is notified | Channel |
|---|---|---|---|---|
| [e.g., Stage submitted] | INFO | — | — | — |
| [e.g., API failure > 3x] | ERROR | Yes | On-call engineer | PagerDuty |

Include: events to instrument, dashboard requirements, SLA breach alerts, and operational notifications.

---

### Risks and tradeoffs

| System / area | Risk | Recommendation |
|---|---|---|
| [e.g., Third-party API] | [e.g., Rate limit on peak load] | [e.g., Async call with retry queue] |

---

### Open questions

| Question | Answer / Decision | Owner | Target date |
|---|---|---|---|
| | | | |

---

### Test cases

Maps 1:1 to every scenario in the acceptance criteria. No scenario may be left untested. No test case may exist without a linked scenario.

**TC-[n] — [short title]**

- Scenario reference: Scenario [n] from acceptance criteria
- Pre-condition: [state that must exist before test execution]
- Test input / trigger: [the action or data that initiates the test]
- Expected result: [exact output, status change, or system behaviour]
- Pass criteria: [what confirms the test has passed]
- Test type: positive / negative / edge case

---

### Appendix

**Backlog:**
Items deferred from this spec for a future phase. Each item includes a brief rationale for deferral.

**References:**
Links to related PRDs, specs, architecture docs, API contracts, or design files.

---

## File output instructions

1. Derive the filename slug: lowercase the spec name, replace spaces with hyphens, strip special characters. Example: "User Authentication Flow" → `user-authentication-flow`.
2. Check if `/specs` exists in the current working directory. If not, create it.
3. Write the completed spec to `/specs/{slug}.md`.
4. Generate the beautified HTML version (see **HTML beautification** section below) and save to `/specs/{slug}.html`.
5. Confirm to the user: "Spec saved to `/specs/{slug}.md` — beautified HTML at `/specs/{slug}.html`."

---

## HTML beautification

After saving the markdown spec, generate a standalone HTML file that renders the spec as a professional, print-ready document using the spec's design system. This is not a raw markdown-to-HTML conversion — it is a designed document.

### UI wireframes (mandatory when spec has UI/UX requirements)

When the spec includes a UI/UX requirements section, the HTML file must include a **UI wireframes** section (`<div class="wireframe-container">`) below the spec document. This section contains interactive, clickable wireframes that show:

1. **Component layout** — exact arrangement of sections, fields, cards, tables, and buttons as they appear in the application
2. **Interactive behavior** — clickable accordions, filter pills, dropdowns, tabs, and toggle states that work in the HTML
3. **Conditional visibility** — demonstrate show/hide behavior (e.g., selecting a dropdown value reveals new sections)
4. **Loading and error states** — skeleton loaders, error badges, progress indicators
5. **Data display patterns** — how API responses, document lists, and computed values render in tables and cards

The wireframes use the same design tokens as the spec document (`:root` variables) but follow the **LOS design system** for component patterns (Inter font, not Tahoma; 12px border-radius cards; left-accent nav). Mark new sections with a green "New" badge and existing sections with a gray "Existing" badge.

The wireframe section must be self-contained and interactive — opening the HTML in a browser should let the reviewer click through the UI flow without any backend or build step. Use vanilla JavaScript for interactivity (no framework dependencies).

### Mandatory HTML boilerplate

Every HTML spec file must open with this exact `<head>` block:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>[Spec name] — Specification</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap">
<style>
:root {
  --color-primary-900: #111827;
  --color-primary-600: #374151;
  --color-primary-400: #6b7280;
  --color-primary-200: #e5e7eb;
  --color-primary-50:  #f9fafb;
  --color-accent-600:  #2563eb;
  --color-accent-100:  #dbeafe;
  --color-success-600: #16a34a;
  --color-success-100: #dcfce7;
  --color-warning-600: #d97706;
  --color-warning-100: #fef3c7;
  --color-error-600:   #dc2626;
  --color-error-100:   #fee2e2;
  --surface-page:      #ffffff;
  --surface-subtle:    #f9fafb;
  --surface-normal:    #f3f4f6;
  --surface-action:    #2563eb;
  --surface-primary:   #eff6ff;
  --surface-success:   #dcfce7;
  --surface-warning:   #fef3c7;
  --surface-error:     #fee2e2;
  --text-heading:      #111827;
  --text-body:         #374151;
  --text-body-light:   #6b7280;
  --text-action:       #2563eb;
  --text-success:      #16a34a;
  --text-warning:      #d97706;
  --text-error:        #dc2626;
  --border-default:    #e5e7eb;
  --border-dark:       #d1d5db;
  --font-body:         'Inter', system-ui, sans-serif;
  --font-mono:         'JetBrains Mono', monospace;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: var(--font-body); background: var(--surface-subtle); color: var(--text-body); font-size: 14px; line-height: 1.6; }
</style>
```

### Document layout CSS

Add these rules inside the same `<style>` block, after the token reset:

```css
/* --- Document layout --- */
.spec-container { max-width: 960px; margin: 0 auto; padding: 32px 24px 64px; }

/* Header card */
.spec-header {
  background: linear-gradient(135deg, var(--color-cobaltblue-900) 0%, var(--color-bluesteel-600) 100%);
  color: #fff; border-radius: 16px; padding: 40px 36px 32px; margin-bottom: 32px;
}
.spec-header h1 { font-size: 28px; font-weight: 700; margin-bottom: 8px; color: #fff; }
.spec-header .spec-meta { display: flex; flex-wrap: wrap; gap: 16px 32px; margin-top: 16px; font-size: 13px; opacity: 0.9; }
.spec-header .spec-meta span { display: inline-flex; align-items: center; gap: 6px; }

/* Status badge in header */
.spec-badge {
  display: inline-block; padding: 4px 12px; border-radius: 20px; font-size: 12px; font-weight: 600;
  background: rgba(255,255,255,0.2); color: #fff; text-transform: uppercase; letter-spacing: 0.5px;
}

/* Section cards */
.spec-section {
  background: var(--surface-page); border: 1px solid var(--border-default); border-radius: 12px;
  padding: 28px 32px; margin-bottom: 20px;
}
.spec-section h2 {
  font-size: 18px; font-weight: 700; color: var(--text-heading); margin-bottom: 16px;
  padding-bottom: 10px; border-bottom: 2px solid var(--color-bluesteel-100);
}
.spec-section h3 { font-size: 15px; font-weight: 600; color: var(--text-heading); margin: 16px 0 8px; }
.spec-section p { margin-bottom: 12px; }
.spec-section ul, .spec-section ol { margin: 8px 0 12px 20px; }
.spec-section li { margin-bottom: 6px; }

/* Tables */
.spec-table { width: 100%; border-collapse: collapse; margin: 12px 0 16px; font-size: 13px; }
.spec-table th {
  background: var(--surface-primary); color: var(--text-heading); font-weight: 600;
  text-align: left; padding: 10px 14px; border-bottom: 2px solid var(--color-bluesteel-200);
  font-size: 12px; text-transform: uppercase; letter-spacing: 0.3px;
}
.spec-table td { padding: 10px 14px; border-bottom: 1px solid var(--border-default); vertical-align: top; }
.spec-table tr:hover td { background: var(--surface-subtle); }

/* Priority badges */
.priority-p0 { display: inline-block; padding: 2px 8px; border-radius: 4px; font-size: 11px; font-weight: 700; background: var(--surface-error); color: var(--text-error); }
.priority-p1 { display: inline-block; padding: 2px 8px; border-radius: 4px; font-size: 11px; font-weight: 700; background: var(--surface-warning); color: var(--text-warning); }
.priority-p2 { display: inline-block; padding: 2px 8px; border-radius: 4px; font-size: 11px; font-weight: 700; background: var(--surface-information); color: var(--text-information); }

/* Status indicators */
.status-complete { color: var(--text-success); }
.status-partial { color: var(--text-warning); }
.status-missing { color: var(--text-error); }

/* Reviewer status badges */
.reviewer-pending { display: inline-block; padding: 2px 10px; border-radius: 12px; font-size: 11px; font-weight: 600; background: var(--surface-warning); color: var(--text-warning); }
.reviewer-approved { display: inline-block; padding: 2px 10px; border-radius: 12px; font-size: 11px; font-weight: 600; background: var(--surface-success); color: var(--text-success); }

/* Test case cards */
.tc-card {
  background: var(--surface-subtle); border: 1px solid var(--border-default); border-radius: 8px;
  padding: 16px 20px; margin-bottom: 12px;
}
.tc-card .tc-title { font-size: 14px; font-weight: 700; color: var(--text-heading); margin-bottom: 8px; }
.tc-card .tc-row { display: flex; gap: 8px; margin-bottom: 4px; font-size: 13px; }
.tc-card .tc-label { font-weight: 600; color: var(--text-body-light); min-width: 120px; flex-shrink: 0; }
.tc-type-positive { border-left: 3px solid var(--color-ironleaf-500); }
.tc-type-negative { border-left: 3px solid var(--color-blastred-500); }
.tc-type-edge { border-left: 3px solid var(--color-amberore-500); }

/* Scenario list */
.scenario-item { padding: 8px 0; border-bottom: 1px solid var(--border-default); }
.scenario-item:last-child { border-bottom: none; }
.scenario-num { font-family: var(--font-mono); font-size: 12px; font-weight: 600; color: var(--text-action); margin-right: 8px; }

/* Confidence footer */
.spec-confidence {
  background: var(--surface-page); border: 1px solid var(--border-default); border-radius: 12px;
  padding: 24px 32px; margin-top: 24px; text-align: center;
}
.confidence-score { font-size: 40px; font-weight: 700; color: var(--text-action); }
.confidence-label { font-size: 13px; color: var(--text-body-light); margin-top: 4px; }

/* Monospace for IDs, codes */
.mono { font-family: var(--font-mono); font-size: 13px; }

/* Insufficient info marker */
.insufficient { font-style: italic; color: var(--text-body-light); background: var(--surface-warning); padding: 2px 8px; border-radius: 4px; font-size: 12px; }

/* Print styles */
@media print {
  body { background: #fff; }
  .spec-container { max-width: 100%; padding: 0; }
  .spec-header { break-after: avoid; }
  .spec-section { break-inside: avoid; border: 1px solid #ddd; }
}

/* Responsive */
@media (max-width: 768px) {
  .spec-container { padding: 16px 12px 48px; }
  .spec-header { padding: 24px 20px; }
  .spec-header h1 { font-size: 22px; }
  .spec-section { padding: 20px 16px; }
  .spec-table { font-size: 12px; }
  .spec-table th, .spec-table td { padding: 8px 10px; }
}
```

### Section-to-HTML mapping

Render each markdown spec section as a `<div class="spec-section">` card. Map these specific patterns:

| Spec section | HTML treatment |
|---|---|
| Header block | `.spec-header` gradient card with metadata row (version, status badge, category, date, linked-to) |
| Reviewer matrix | Table with `.reviewer-pending` / `.reviewer-approved` badges per row |
| Abbreviations | Two-column `.spec-table` — term in `<strong>`, definition in normal weight |
| Executive summary | Single `<p>` block inside a section card — no sub-headings |
| Problem statement | Bulleted list or prose paragraphs. Data references in `<em>` |
| Goals / OKRs | OKR table with `.spec-table`. Success metrics as an ordered list. Definition of done as a checklist with checkbox marks |
| Scope | Two sub-sections (in-scope / out of scope) each as `<ul>` inside the same card |
| Actors | `.spec-table` with 4 columns |
| Functional requirements | `.spec-table` with priority badges (`.priority-p0`, `.priority-p1`, `.priority-p2`). User story column in italics |
| Acceptance criteria | Scenarios as `.scenario-item` divs with `.scenario-num` prefix. Business rules as a sub-table |
| NFRs | `.spec-table` with dimension column bolded |
| System architecture | Two sub-tables: systems involved + data model impact |
| UI/UX requirements | Prose or list. Mark "no UI component" in `.insufficient` style if backend-only |
| Error handling | `.spec-table` with 5 columns. Failure mode column in `<strong>` |
| Monitoring | `.spec-table` with alert trigger column using color tokens (ERROR = red, WARN = amber, INFO = blue) |
| Risks | `.spec-table` — risk column in `<strong>`, recommendation in normal |
| Open questions | `.spec-table` — unanswered rows get `.insufficient` badge in answer column |
| Test cases | `.tc-card` cards with left-border color by type (positive=green, negative=red, edge=amber) |
| Confidence score | `.spec-confidence` footer card with large score number |

### UX content rules for HTML output

Apply these to every string in the HTML — same rules as the markdown, but enforced in rendered output:

1. **Sentence case** — all headings, labels, table headers, card titles. No title case on general text. Exceptions: abbreviations (DSCR, PAN, etc.) and domain-specific proper nouns
2. **No trailing full stops** on labels, headings, or table cells
3. **Pricing** — `₹X,XX,XXX.XX` format: no space after ₹, Indian numbering, always 2 decimal places
4. **Dates** — `DD MMM YYYY HH:MM:SS AM/PM` format. No ISO 8601
5. **Colons** — no space before, one space after: `Status: Active`
6. **Alternatives** — intentional whitespace around `/`: `NEFT / RTGS / IMPS`
7. **No underlined text** — no `<u>` tags, no `text-decoration: underline`
8. **No raw hex** in rendered styles — all colors via `var(--token-name)`
9. **Monospace** for IDs, codes, and version numbers — `class="mono"`

### HTML quality check

Before saving the HTML file, verify:

1. Full `:root` token block + Inter + JetBrains Mono in `<head>` — no missing fonts
2. Every section from the markdown spec has a corresponding `.spec-section` card
3. No raw hex in any `style` attribute or inline CSS — all via tokens
4. All tables use `.spec-table` class — no unstyled tables
5. Priority badges rendered as `.priority-p0` / `.priority-p1` / `.priority-p2` spans
6. Test cases rendered as `.tc-card` with correct type border color
7. Sentence case on all rendered text (re-check every heading and label)
8. Prices and dates follow the format rules — verify after rendering
9. File opens in a browser and looks fully designed — no unstyled text, no missing colours, no broken layout
10. Print-friendly — `@media print` rules present, section cards don't break mid-page

---

## Review mode

When the user provides an existing spec, document, or partial draft, audit it against each section above:

| Section | Status | Gap / Issue |
|---|---|---|
| Header block | ✅ / ⚠️ / ❌ | |
| Reviewer matrix | ✅ / ⚠️ / ❌ | |
| Abbreviations | ✅ / ⚠️ / ❌ | |
| Executive summary | ✅ / ⚠️ / ❌ | |
| Problem statement | ✅ / ⚠️ / ❌ | |
| Goals and OKRs | ✅ / ⚠️ / ❌ | |
| Scope | ✅ / ⚠️ / ❌ | |
| Actors | ✅ / ⚠️ / ❌ | |
| Functional requirements | ✅ / ⚠️ / ❌ | |
| Acceptance criteria | ✅ / ⚠️ / ❌ | |
| NFRs | ✅ / ⚠️ / ❌ | |
| System and data architecture | ✅ / ⚠️ / ❌ | |
| UI/UX requirements | ✅ / ⚠️ / ❌ | |
| Error handling | ✅ / ⚠️ / ❌ | |
| Monitoring | ✅ / ⚠️ / ❌ | |
| Risks and tradeoffs | ✅ / ⚠️ / ❌ | |
| Open questions | ✅ / ⚠️ / ❌ | |
| Test cases | ✅ / ⚠️ / ❌ | |
| Definition of done | ✅ / ⚠️ / ❌ | |

Legend: ✅ Complete | ⚠️ Partial — needs expansion | ❌ Missing entirely

After the gap report: "I can fill the missing sections now — provide the needed inputs, or I'll mark them as insufficient."

Save the updated spec back to `/specs/{filename}.md` (overwrite draft, bump version to 1.1).

---

## Quality check before output

Before finalising, verify:

**Markdown quality:**
- Every section is populated or explicitly marked as insufficient.
- Requirements table has no bundled requirements (one row = one requirement).
- Every acceptance criteria scenario has exactly one test case.
- NFRs cover at minimum: performance, security, scalability, reliability.
- Error handling covers every negative scenario.
- Definition of done is present and actionable.
- All content is in sentence case.
- File has been saved to `/specs/{slug}.md`.
- Confidence score is appended as the last line.

**HTML quality:**
- Full token block + Inter + JetBrains Mono loaded in `<head>`.
- Every markdown section rendered as a `.spec-section` card.
- No raw hex in styles — all via `var(--token-name)`.
- Tables use `.spec-table`, priorities use `.priority-*` badges, test cases use `.tc-card`.
- Sentence case on all rendered text. Prices in `₹X,XX,XXX.XX`. Dates in `DD MMM YYYY`.
- Opens in browser with full design — no unstyled elements.
- File has been saved to `/specs/{slug}.html`.

---

## Confidence score

Append to the end of every completed spec. Not during Q&A. Not during review gap reports.

```
Confidence score: [0–100]%
Basis: [one sentence on what's solid vs. what's uncertain]
```

If below 80%: list specifically which sections carry low confidence and why.
