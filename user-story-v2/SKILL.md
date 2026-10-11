---
name: user-story-v2
description: Generate or review comprehensive user stories in the standard v2 format. Use this skill when a PM provides a raw feature idea, a partial story, or an existing ticket for review. The model must never assume missing information — it asks all clarifying questions upfront in a single batch before generating output. Output includes a fully structured user story followed by a 1:1 mapped test case list.
---

# User Story V2 — Skill Guide

This skill governs how the model generates or evaluates user stories in the standardised v2 format. It applies to both modes:

- **Generate mode**: A PM provides a raw feature idea or rough description. The model structures it into a complete v2 user story.
- **Review mode**: A PM provides an existing story or ticket. The model evaluates completeness against the v2 standard and fills gaps or flags missing sections.

---

## Behaviour rules

**Rule 1 — Ask first, write never.**
Before generating any story output, the model must identify every missing, ambiguous, or assumable piece of information and ask all clarifying questions in a single upfront batch. Do not generate partial output and ask later. Do not assume and proceed.

**Rule 2 — Sentence case only.**
All section content must be written in sentence case. No all-caps headings, no title-casing body text, no excessive bold outside of field labels.

**Rule 3 — No filler.**
Do not populate sections with generic placeholder text. If information is insufficient after the Q&A round, mark the field explicitly as: `[ Insufficient information — owner to provide ]`

**Rule 4 — Test cases map 1:1 to acceptance criteria.**
Every scenario defined in the acceptance criteria section must have exactly one corresponding test case entry in the test case list at the end of the document. No orphan test cases. No untested scenarios.

**Rule 5 — Dual-mode awareness.**
If the input is a raw idea → generate from scratch after Q&A.
If the input is an existing story → review against the v2 checklist, call out gaps, and offer to fill them.

---

## Upfront clarification protocol

When the model receives a feature input, it must scan for gaps across all sections below and batch every question into a single numbered list before proceeding. The question set must cover at minimum:

1. Who is the primary actor — a user, a system, or both?
2. What is the trigger for this story — a user action, a scheduled event, an API call, or a system state change?
3. What is the exact expected output or end state when everything works correctly?
4. What are the known negative or edge case scenarios the story must handle?
5. Are there any hard business rules or domain constraints that govern this feature (e.g., skip logic, status thresholds, queue priorities)?
6. Which external systems, APIs, or internal services does this story interact with — and who owns them?
7. Which database tables or data objects are created, read, updated, or deleted?
8. Are there non-functional requirements — performance SLAs, concurrency limits, data retention rules, security constraints?
9. Is there a UI/UX component, or is this a backend/system story only?
10. Does this story touch any regulatory, compliance, or product governance requirements?
11. What does "done" look like — what specific deliverables and validations must be complete before this story closes?
12. Are there monitoring, alerting, or notification requirements tied to this story?
13. Are there rollback or recovery provisions if the feature fails in production?

The model must add domain-specific follow-up questions beyond this baseline where context demands it.

---

## Output structure

Once all questions are answered, the model generates the full story in the following structure. Every section header is a label; every content block is in sentence case prose or structured lists as appropriate.

---

### Section 1 — Overall product requirements

**System story**
A single concise statement of what this story delivers and why it exists within the product. Written from the system or user perspective as appropriate.

**Core assumptions**
List the assumptions that must hold true for this story to be implementable. If an assumption is violated, the story breaks. Each assumption is a standalone sentence.

**Pre-conditions**
Conditions or system states that must exist before this story can execute. These are hard gates, not preferences.

**Key considerations**
Constraints, trade-offs, priorities, or design decisions that the development team must account for. Include timing conflicts, data integrity risks, dependency sequencing, and performance sensitivities.

**KPIs (product, technical, or user)**
Measurable indicators that define whether this story has achieved its intended impact. Include at least one product KPI and one technical KPI where applicable.

---

### Section 2 — Acceptance criteria

**Expected behaviour**
List every scenario the system must handle — positive flows, negative flows, and edge cases. Each scenario follows this structure:

- Scenario [number]: [condition] → [expected system output or state change]

Do not leave any scenario implicit. If a business rule applies, cite it in the scenario.

**Business rules validation**
Domain logic the system must enforce. Each rule is written as a declarative constraint tied to the domain model. Examples: skip conditions, status gate logic, queue priority rules, threshold enforcement.

**Definition of done**
An explicit checklist. The story is only closeable when every item is checked. Must include: code reviewed, unit tests passing, integration tests passing, UAT sign-off, documentation updated, feature demonstrated to stakeholders, logs implemented for all failure and success states.

**Non-functional requirements (NFRs)**
System-wide quality requirements. Cover: response time SLAs, throughput or concurrency limits, data security and access control, scalability assumptions, and usability standards where a UI exists.

**Error handling and rollback requirements**
For every failure mode identified in expected behaviour, define: what the system does on failure, how the error is logged, whether a retry mechanism applies, and what the rollback state is if deployment fails. Rollback criteria must specify the trigger condition and the stable state to revert to.

**UI/UX impact**
Describe how the interface must behave to meet user expectations — feedback messages, loading states, empty states, error displays, confirmation flows. If this is a backend-only story, state that explicitly.

**Monitoring and alerts**
Define what must be logged, what triggers a notification, who is notified, through which channel, and at what threshold. Include both success confirmations and failure escalations.

**Compliance and standards check** *(non-mandatory — include only if applicable)*
Identify any regulatory, legal, or internal governance standards this story must adhere to. Reference the specific regulation or policy by name. If not applicable, omit this field entirely.

---

### Section 3 — Cross-system impact

**Impact across systems and owners**
List every system this story interacts with. For each: system name, nature of interaction (read / write / trigger / receive), API or integration method, and system owner name.

**Impact across tables and data objects**
List every table or data entity affected. For each: table name, operation performed (create / read / update / delete), and the field or fields modified.

---

### Section 4 — Functional checklist

This section is designed for business users to review and confirm adherence without accessing the ticketing system. It can be shared over email as a standalone reference.

**Positive flow**
Step-by-step system behaviour for the primary success path. Each step states the action and the resulting system state or output. Written in plain language.

**Negative flow**
Step-by-step system behaviour for each failure or exception path. Each step states the failure condition, the system response, and the end state (e.g., error logged, status updated, manual intervention flagged).

**Sample working / reference ticket** *(optional)*
If a reference ticket exists, link it here with a brief description of what it demonstrates.

---

### Section 5 — Test case list

This section maps 1:1 to every scenario listed under "Expected behaviour" in the acceptance criteria. Each test case entry follows this structure:

**TC-[number] — [short test case title]**

- Scenario reference: Scenario [number] from acceptance criteria
- Pre-condition: [state that must exist before test execution]
- Test input / trigger: [the action or data that initiates the test]
- Expected result: [the exact output, status change, or system behaviour the tester must observe]
- Pass criteria: [what confirms the test has passed]
- Test type: [positive / negative / edge case]

No scenario from the acceptance criteria may be left without a corresponding test case. No test case may exist without a linked scenario.

---

## Quality check before output

Before finalising output, the model must verify:

- Every section is populated or explicitly marked as insufficient.
- Every acceptance criteria scenario has exactly one test case.
- No assumptions have been silently baked into any field.
- All content is in sentence case.
- NFRs cover at minimum: performance, security, and scalability.
- Error handling covers every negative scenario in expected behaviour.
- The functional checklist is self-contained enough to be understood without reading the full story.

Confidence score: The model must end its output with a confidence score (0–100%) reflecting the completeness of information provided by the PM. If the score is below 80%, explicitly state which sections carry uncertainty and why.


---

## Next skill

For a full PRD-level spec that includes stories and test cases, use `/spec-creator`. Once stories are approved, run `/tech-development-plan`.
