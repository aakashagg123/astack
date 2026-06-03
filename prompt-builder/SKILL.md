---
name: "prompt-builder"
description: >
  Comprehensive prompt engineering skill — use whenever you need to build a prompt for a recurring workflow (product docs, decision frameworks, comp analysis, research, code specs, stakeholder comms, or scenario planning). Also trigger when you want to improve or audit an existing prompt, or when a task seems too broad/vague and needs structured prompt scaffolding before execution.
---

# Prompt Builder (v2)

## Operating Philosophy

Prompts are **leverage artifacts** — not instructions. A great prompt codifies the builder's mental model so any LLM replicates their thinking at scale, cold, without context-setting. Write once. Use many times. Zero degradation.

The standard: every prompt must pass the **cold-hand test** — hand it to a competent stranger with zero context and the output should be indistinguishable from what an expert practitioner would produce.

---

## Step 1: Decompose the Request

Classify before writing a word.

### Domain
| Domain | Examples |
|---|---|
| **Product Strategy** | PRDs, LOS specs, Customer Master, feature logic, build-vs-buy |
| **Decision Framework** | Scenario planning, tradeoff matrices, make-or-break analysis |
| **Stakeholder Comms** | CEO/CTO briefs, Slack drafts, escalation emails, mandate formalization |
| **Investment Research** | Sector analysis, thesis building, competitive frameworks, valuation math |
| **Hiring/Evaluation** | PM candidate scoring, interview rubrics, culture-fit assessment |
| **AI/LLM Tooling** | Workflow automation, use case identification, demo scripting |
| **Compensation/Career** | Offer analysis, leverage mapping, scenario probability weighting |
| **Data/Analytics** | Defect analysis, adoption tracking, KPI dashboards |
| **Learning/Frameworks** | Curriculum design, mental model building, concept synthesis |

### Output Type
- **One-shot** → Single response (report, email, analysis)
- **Iterative** → Multi-turn with checkpoints
- **Template** → Reusable with `[VARIABLE]` slots
- **Agent** → Autonomous task with tools

### Deployment Context ← NEW
Where will this prompt run? This determines architecture.

| Context | Implication |
|---|---|
| **Direct Claude chat** | Single monolithic prompt; user provides input inline |
| **API / product integration** | Split into system prompt + user turn |
| **Workflow automation** | Needs rigid output format; often JSON or structured markdown |
| **One-time analysis** | Can be verbose; context baked in |
| **Recurring template** | Must be parameterized; strip all hardcoded content |

---

## Step 2: Extract Requirements

```
CONTEXT:       What situation is this solving?
GOAL:          Success in one sentence — what does the output accomplish?
INPUT FORMAT:  What does the user provide? (doc, list, question, raw data?)
OUTPUT FORMAT: Structured? Prose? Table? Length target?
PERSONA:       Specific expert role — not generic
CONSTRAINTS:   What must it NOT do?
TONE:          Hemingway? McKinsey? Feynman? Match to audience.
DEPLOYMENT:    Chat / API / template / workflow?
SIGNAL MAP:    Needs decomposition? Scenarios? Confidence scores?
```

If goal or output format is ambiguous — ask **one clarifying question**. Never guess.

---

## Step 3: System vs. User Prompt Split ← NEW

For deployed prompts (API, workflows, product integrations), the split is architectural — not cosmetic.

### System Prompt — What Never Changes
- Expert persona and role definition
- Tone, style, negative constraints
- Output format specification
- Domain vocabulary and context
- Processing rules (signal-map logic, confidence scoring)
- What to always do / never do

### User Turn — What Changes Per Use
- The actual task or question
- Dynamic inputs (the document, the data, the decision)
- Variable context specific to that invocation

### Split Template
```
SYSTEM:
You are [SPECIFIC EXPERT ROLE] at [ORGANIZATION TYPE].

Your operating rules:
- [RULE 1]
- [RULE 2]
- [TONE + CONSTRAINTS]

Output format for every response:
[FORMAT SPEC]

---

USER:
[DYNAMIC INPUT]

Task: [SPECIFIC ASK FOR THIS INVOCATION]
```

**For direct Claude chat:** Combine into one prompt. System rules go first, user input goes last, separated by `---`.

---

## Step 4: Select Architecture

### Architecture A — Direct Instruction
**Best for:** One-shot tasks with clear inputs/outputs. Simplest. Use by default unless the task is complex.

```
You are [SPECIFIC EXPERT ROLE — e.g., "Senior credit analyst at a lending-focused financial institution"].

Context: [2-3 sentences. Situation only. No backstory.]

Task: [One sentence. What to do.]

Input: [What the user will paste below the prompt]

Output:
- Format: [prose / table / numbered list / JSON]
- Length: [under 300 words / one page / etc.]
- Tone: Direct. No hedging. No filler. Hemingway-Orwell standard.
- Exclude: [specific things to omit]

[OUTPUT EXAMPLE — even a skeleton anchors the format:]
Example output structure:
## [Section]
[1-2 lines showing expected depth and style]
```

---

### Architecture B — Signal-Map Framework
**Best for:** Complex decisions, scenario planning, strategic analysis. Forces decomposition before conclusion.

```
You are [ROLE]. Apply systematic decomposition before answering.

Processing rules — execute in this order:
1. Classify the question: tactical, strategic, validation, or framework-building?
2. Identify stress signals: vagueness, missing deadlines, absent written directives
3. Decompose into components before synthesizing
4. If a decision is in play: map probability-weighted scenarios (not point estimates)
5. End every response with a confidence score (0–100%) and explicit reasoning

Context: [SITUATION]

Input: [USER PROVIDES]

Output format:
**Decomposition:** [What is actually being asked?]
**Analysis:** [Domain-specific reasoning]
**Scenarios:** [If applicable — each with probability %]
**Next move:** [Forward-looking. Not "summary of situation." What to do Monday.]
**Confidence:** X% — [one sentence on why]
```

---

### Architecture C — Repeatable Template / Scaffold
**Best for:** PRDs, candidate evaluations, investment screens, defect analyses — anything run more than once.

```
You are [ROLE].

For each [INPUT UNIT — e.g., "feature request", "candidate profile", "stock thesis"], generate the following sections in order:

## Problem Statement
[Instruction: 2-3 sentences max. What pain, whose pain, at what scale.]

## Signal Assessment
[Instruction: What evidence supports this? What's missing?]

## Options
[Instruction: 2-3 alternatives minimum. Never one option.]

## Recommendation
[Instruction: One option, explicitly chosen. State the trade-off accepted.]

## Open Questions
[Instruction: What must be resolved before execution? Max 5.]

Style: Direct. Precise. No bullet soup. Sections in prose unless data demands a table.
Length: [Target word count per section]

Variables (fill before use):
- [INPUT_UNIT]: 
- [CONTEXT]:
- [CONSTRAINT]:
```

---

### Architecture D — Adversarial / Pre-Mortem
**Best for:** Pressure-testing decisions, stress-testing plans, surfacing blind spots before commitment.

```
You are a sharp, skeptical [EXPERT — e.g., "NBFC credit risk officer who has seen 50 failed lending tech implementations"].

Your job is to find everything wrong with what follows. Not balanced feedback. Not "here's what's good and bad." Only flaws.

Find:
- Logical errors and internal contradictions
- Assumptions stated as facts
- Probability miscalibrations (overconfidence, underweighting tail risks)
- Missing stakeholders or missing data
- Execution gaps between strategy and reality

Do NOT validate. Do NOT reassure. Do NOT suggest improvements unless asked.

Input: [PLAN / ARGUMENT / DECISION]

Output:
- Ordered list of concerns — most dangerous first
- Each concern: one sentence problem statement + one sentence why it matters
- Final line: "The single most dangerous assumption is: [X]"

Length: As short as possible. One concern per line. No padding.
```

---

### Architecture E — Stakeholder Comms
**Best for:** CEO/CTO emails, mandate formalization, Slack strategy, escalation memos. Gracián-encoded.

```
You are a strategic communications advisor to a Senior PM at a fast-scaling NBFC.

Operating rules:
- Gracián principle: document and deliver — don't negotiate, don't perform
- Open with the move, not the backstory
- State the ask or update in the first sentence
- Close with a specific checkpoint or next step
- Zero desperation signals. Zero emotional framing. Zero vague asks.
- Audience: [CEO / CTO / Direct Manager / Peer]
- Tone: [Direct-strategic / Diplomatic-firm / Formal / Casual-strategic]

Input: [RAW CONTEXT — bullets or prose, doesn't matter]

Output: [EMAIL with subject / SLACK message / MEMO — specify]
Length: [Under 150 words for Slack. Under 300 for email. Under 500 for memo.]

[OUTPUT EXAMPLE — anchors tone and structure:]
Example email:
Subject: LOS Vision Scope — Q4 FY26 Confirmation

Hi [Name], following our discussion, wanted to confirm the scope in writing:
[2 sentences of mandate]. Happy to align on sequencing in our next 1:1.
```

---

## Step 5: Output Anchoring ← NEW

The single most underused technique in prompt engineering. Show — don't just specify.

After every output format spec, add a **skeleton example** that demonstrates:
- The depth expected (not content — just structure and length signal)
- The prose style (not template language — actual register)
- What "done" looks like

### How to write an anchor:
```
Output example (structure only — not the actual answer):

## Risk Assessment
Third-party API dependency accounts for 51% of defects. Root cause is [specific mechanism], 
not [common misdiagnosis]. Mitigation requires [specific action], not [common but wrong action].

## Recommendation
Do X by [date]. Not Y — here's why that fails: [one sentence].

Confidence: 78% — missing data on [specific gap] limits certainty.
```

**Rule:** The anchor should be 10-20% of the expected full output length. Long enough to show style and depth. Short enough to not contaminate the actual response.

---

## Step 6: Anti-Pattern Library ← NEW

The fastest way to build prompting skill is recognizing failure before it ships. Eight patterns, each with before/after.

Read `references/anti-patterns.md` for full annotated examples.

Summary of the eight:

| # | Anti-Pattern | Signal | Fix |
|---|---|---|---|
| 1 | **Vague persona** | "You are an expert" | Specify role, org type, experience level |
| 2 | **Instruction conflict** | Two directives contradict | Audit for conflicts; last instruction wins by default — dangerous |
| 3 | **Missing negative constraints** | Output has filler, hedging, bullets | Add explicit "do NOT" list |
| 4 | **Underspecified format** | Output length/structure varies wildly | Add output anchor example |
| 5 | **Overconstrained length** | Truncated analysis, missing nuance | Replace word count with "as short as complete" |
| 6 | **Persona collapse** | Claude drops role mid-response | Add role reminder mid-prompt; use system prompt for multi-turn |
| 7 | **Scope creep** | Output drifts into adjacent territory | Add "Scope boundary: only address X" |
| 8 | **Context bleed** | Prior conversation overrides prompt | Use explicit "Ignore prior conversation. Start fresh." for clean-slate tasks |

---

## Step 7: Failure Mode Taxonomy ← NEW

When a prompt fails, diagnose before rewriting. Prompts fail in five ways:

### F1 — Instruction Conflict
Two directives contradict each other. Claude picks one arbitrarily — usually not the one you wanted.
**Symptom:** Output inconsistently follows one rule but not another.
**Diagnosis:** Read the prompt aloud. Do any two sentences say opposite things?
**Fix:** Explicit priority order. "If X and Y conflict, X wins."

### F2 — Length Miscalibration
**Too loose:** Output bloats with filler. **Too tight:** Output truncates before complete.
**Symptom:** Response either reads like padding or feels cut off.
**Diagnosis:** No length spec, or word count without format spec.
**Fix:** Specify both length target AND "as short as complete" as the override rule.

### F3 — Persona Collapse
Claude opens in role, then drifts back to default assistant behavior.
**Symptom:** Response starts sharp, becomes hedged and cautious 2/3 through.
**Diagnosis:** Role only defined at the top; no reinforcement; long prompt dilutes it.
**Fix:** Move persona + negative constraints to a system prompt. Add a closing reminder: "Stay in role. Do not soften conclusions."

### F4 — Scope Creep
Output addresses adjacent topics the prompt didn't ask for.
**Symptom:** "I also thought it worth noting that..." sections appear.
**Diagnosis:** Goal statement too broad; no scope boundary defined.
**Fix:** Add "Scope boundary: address only [X]. Do not discuss [Y] unless explicitly asked."

### F5 — Context Bleed
Prior conversation context overrides prompt instructions.
**Symptom:** Output references earlier conversation content that conflicts with the prompt.
**Diagnosis:** Prompt deployed mid-conversation with established context.
**Fix:** Open with "Ignore prior conversation. Treat this as a fresh task." For templates used repeatedly, use the API with a clean context window.

---

## Step 8: Quality Gate

Before delivering any prompt, run this checklist. No exceptions.

```
□ Cold-hand test: could a stranger use this cold and produce expert-grade output?
□ Persona: specific role, org type, implied experience level?
□ Negative constraints: explicit "do NOT" list present?
□ Output format: format + length + tone all specified?
□ Output anchor: skeleton example present?
□ Failure mode check: scanned for F1–F5?
□ Anti-pattern check: none of the eight present?
□ Deployment split: system vs. user turn separated if API/workflow context?
□ Parameterized: all hardcoded content replaced with [VARIABLES]?
□ Signal-map aligned: decomposition, scenarios, confidence score where needed?
```

Score: 10/10 required. Fix any gap before presenting.

---

## Step 9: Present the Output

```
### [PROMPT NAME]
> One-line: what it does, when to use it.

**SYSTEM PROMPT** (if API/workflow):
[prompt text]

**USER TURN** (if API/workflow):
[prompt text]

— or —

**PROMPT** (if chat):
[full prompt text]

---
Architecture: A / B / C / D / E
Deployment: Chat / API / Template / Workflow
Variables to fill: [list]
Failure modes guarded: [F1–F5 list]
Confidence: X%
```

---

## Builder defaults

Baked in unless explicitly overridden:

| Default | Value |
|---|---|
| Tone | Hemingway clarity + Orwell precision |
| Sugar-coating | Never |
| Bullet overuse | Prose preferred; bullets only when structural |
| Decision outputs | Probability-weighted scenarios, not point estimates |
| Confidence scoring | Always 0–100% with explicit reasoning |
| Forward framing | "Next move" — never "summary of what happened" |
| Stress signals | Flag vagueness, missing deadlines, absent written formalization |
| Brevity | "As short as complete" overrides any word count |
| Output anchoring | Skeleton example in every prompt |

---

## Reference Files

Load only when needed:

- **`references/anti-patterns.md`** — Full annotated before/after for all 8 anti-patterns
- **`references/few-shot-examples.md`** — Worked examples for each architecture (A–E)
- **`references/domain-templates.md`** — Domain-specific starter prompts: PRDs, investment screens, hiring rubrics, comp analysis, comms

---

## Meta-Rule

Every prompt produced must pass one test: would a domain expert read it and say *"this thinks like me"*?

If no — rebuild. Don't patch.

*Last updated: Feb 2026 — v2*
