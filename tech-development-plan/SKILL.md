---
name: tech-development-plan
description: "Generates a comprehensive, executable technical development plan from a PRD and user stories. Use immediately after /prd-generator and /user-story-v2 have been run — or whenever a PM or tech lead provides a PRD, user stories, or acceptance criteria and needs a sequenced implementation blueprint. Trigger on: 'create a dev plan', 'build a tech plan', 'plan this feature', 'how do we implement this', 'sequence the implementation', 'give me a development plan', 'create implementation plan', '/tech-development-plan', or any request to translate requirements into an ordered engineering execution plan. Also trigger proactively after user stories are finalised in a conversation — the natural next step is always the technical plan. Output is editable markdown submitted for human approval before execution begins."
---

# Tech Development Plan — Skill Guide

Translates a finalised PRD and user story set into a sequenced, executable technical
development plan. This is the bridge between *what to build* (PRD) and *how to build it*
(code). The plan is submitted for human approval before any execution begins.

Two modes:
- **Generate mode**: PRD + stories provided → produce full plan
- **Review mode**: Existing plan provided → audit completeness, flag gaps, offer to fill

---

## Position in workflow

```
/prd-generator  →  /user-story-v2  →  /tech-development-plan  →  [APPROVAL GATE]
                                                                        ↓
                                              /secure-coding-practice + /red-green-tdd
```

This skill is the last step before execution. Nothing gets coded until this plan is
approved. If the plan is rejected, it is revised here — not in the code.

---

## Entry gates

**Gate 1 — Scope contract.** If the user's invocation or approval is vague or short (≤ 5 words — "plan it", "go ahead", "yes"), do NOT start. Echo a one-line contract and wait:

> → Contract: I'll plan [feature] from [PRD / stories], producing a phased plan for approval. Confirm or adjust.

Bypass only on an explicit, scoped instruction.

**Gate 2 — First-approach checkpoint.** The plan encodes the architecture — so before fleshing out every phase, state the core approach in ≤ 2 lines and confirm it:

> Approach: [architecture / pattern, which modules, build sequence].

This catches a wrong architectural framing in 10 seconds instead of after a full plan is written. After one rejection, propose a fundamentally different approach; after two, ask the user how they would structure it. A rejected approach is dead for the session.

---

## Behaviour rules

**Rule 1 — Inputs before output.**
Never generate a plan without at least one of: a PRD, a set of user stories, or a
feature brief with acceptance criteria. Ask for missing inputs in a single batch.

**Rule 2 — Trace everything to source.**
Every module, task, and data contract in the plan must trace to a specific story ID
or PRD requirement row. No invented scope. No gold-plating.

**Rule 3 — Sequence is explicit.**
Every task has a dependency chain. Parallelisable work is marked as parallel. Blocked
work is marked as blocked with the specific blocker named.

**Rule 4 — Plan is editable before execution.**
Output is structured markdown. After generating, explicitly state: "Review this plan.
Edit any section. Confirm GO when ready." Do not proceed to code generation
without explicit approval.

**Rule 5 — Risks surface here, not in code reviews.**
Every integration risk, data contract assumption, and third-party dependency is
flagged in the plan — not discovered mid-implementation. The plan is where risks are
resolved cheaply.

**Rule 6 — One task, one owner, one output.**
Each task in the plan is atomic: single assignable unit, defined input, defined
output, defined done criterion. No compound tasks. No ambiguous deliverables.

**Rule 7 — NFRs are first-class in the plan.**
Performance SLAs, security constraints, and regulatory requirements from the PRD/stories
are mapped to specific tasks or cross-cutting concerns — not left as aspirational notes.

---

## Intake protocol

Scan the context for the following. Ask for anything missing in one batch:

1. Is the PRD available? (paste or confirm it's in context)
2. Are the user stories and acceptance criteria available? (paste or confirm)
3. What is the tech stack? (language, framework, DB, cloud, queue system)
4. Are there existing modules this plan extends, or is it greenfield?
5. Who are the engineers? (names/roles, or just team size and seniority mix)
6. What is the target delivery timeline or sprint structure?
7. Are there hard external dependencies? (third-party APIs, other team deliverables)
8. Are there infrastructure or deployment constraints? (CI/CD pipeline, env restrictions)
9. What environments exist? (dev, staging, prod — and promotion process)
10. Any known technical debt in the affected area that the plan must route around?

If PRD and stories are fully in context, skip to plan generation — do not re-ask
questions already answered in those documents.

---

## Output structure

Generate in this exact section order. All content in sentence case.

---

### Header block

```
Feature:         [feature name from PRD]
Plan status:     Draft — pending approval
Plan author:     [default to user if not specified]
PRD reference:   [PRD status + date, or link]
Story set:       [story count + IDs if available]
Target stack:    [language · framework · DB · infra]
Last updated:    [today's date]
```

---

### Plan summary

Two paragraphs max. Readable by a non-engineering stakeholder.

Para 1: What is being built and why. Pulls from PRD executive summary.
Para 2: How the plan is structured — number of phases, key dependencies, estimated
         sequence. Do not include implementation detail here.

---

### Scope confirmation

Explicit restatement of what this plan covers and what it does not. Derived directly
from PRD in-scope / out-of-scope. If any story falls outside PRD scope, flag it here.

**In scope for this plan:**
- [item 1 — maps to PRD requirement or story ID]
- [item 2]

**Out of scope / deferred:**
- [item — reason for deferral, linked PRD backlog item if applicable]

**Scope conflicts identified:**
List any discrepancy between PRD scope and story scope. Each conflict needs resolution
before the plan is approved.

---

### Architecture overview

High-level description of what gets built. Not code — structural decisions.

**New components / modules:**
For each new module: name, responsibility boundary, interface contract (what it
accepts, what it returns), and which stories it serves.

**Modified components:**
For each existing module being changed: what changes, what stays the same, and the
blast radius of the change (other systems potentially affected).

**Data model changes:**
For each table or entity affected (from user story cross-system impact section):
- Table name
- Operation: create / alter / read-only
- Fields added / modified / deprecated
- Migration strategy (if altering existing table with live data)

**API contracts:**
For each new or modified endpoint:

```
Method:     GET / POST / PUT / PATCH / DELETE
Endpoint:   /api/v[n]/[resource]
Auth:       [Bearer token / API key / none]
Request:    { field: type, ... }
Response:   { field: type, ... }
Errors:     [4xx/5xx codes and their meaning]
Story ref:  [story ID(s) this serves]
```

**Integration points:**
For each third-party API or internal service dependency:
- Service name and owner
- Interaction type (sync / async / webhook)
- Failure mode and handling strategy
- Rate limits or quota constraints known

---

### Implementation phases

Break the work into ordered phases. Each phase is independently reviewable and
deployable where possible. Phase 1 always contains the foundation that all later
phases depend on.

For each phase:

#### Phase [N] — [short phase title]

**Objective:** One sentence on what this phase delivers.
**Stories covered:** [list story IDs]
**Depends on:** [prior phase or external dependency, or "none"]
**Estimated effort:** [S / M / L / XL per task, or points if team uses them]
**Can be parallelised:** [yes / no — if yes, which tasks]

| Task ID | Task description | Story ref | Input | Output | Effort | Depends on | Owner |
|---|---|---|---|---|---|---|---|
| T[N.1] | [atomic task — verb + noun] | [story ID] | [what this task consumes] | [what it produces] | S/M/L | — | [role] |
| T[N.2] | | | | | | T[N.1] | |

**Phase done criteria:** Explicit checklist. Phase is complete only when all items
are checked:
- [ ] All tasks in phase complete and reviewed
- [ ] Unit tests passing for all new logic
- [ ] Integration tests passing for all new endpoints
- [ ] Data migrations validated in staging
- [ ] No regression in existing functionality
- [ ] Phase sign-off by [role]

---

### Cross-cutting concerns

Work that spans all phases and is not owned by a single task.

**Security implementation:**
Map each security requirement from PRD/stories to a concrete implementation task.
Reference `/secure-coding-practice` for execution.

| Requirement | Implementation approach | Phase | Owner |
|---|---|---|---|
| Input validation | [e.g., zod schema on all API inputs] | Phase 1 | |
| Auth/authz | [e.g., JWT middleware on all protected routes] | Phase 1 | |
| PII handling | [e.g., mask before logging, encrypt at rest] | Phase 1 | |
| [regulatory ref] | [e.g., DPDP Act consent capture on new data fields] | Phase 1 | |

**Test strategy:**
Derived from story test cases (Section 5 of user-story-v2 output). Reference
`/red-green-tdd` for execution discipline.

| Test type | Coverage target | Tooling | Owner |
|---|---|---|---|
| Unit tests | All non-trivial functions | [framework] | |
| Integration tests | All API endpoints | [framework] | |
| E2E tests | Critical user journeys | [framework] | |
| Load tests | Endpoints with perf SLAs | [framework] | |

**Observability:**
| What to log | Level | Alert threshold | Channel | Owner |
|---|---|---|---|---|
| [event] | INFO/WARN/ERROR | [threshold] | [Slack/PagerDuty/etc] | |

**Database migration plan:**
If any existing tables are modified: migration script approach, rollback script,
zero-downtime strategy, staging validation steps, and prod promotion checklist.

---

### Dependency map

Visual or tabular representation of task dependencies across all phases.

| Task | Blocked by | Blocks | Can parallelise with |
|---|---|---|---|
| T1.1 | — | T1.2, T2.1 | T1.3 |
| T1.2 | T1.1 | T2.2 | — |

Flag any task with no dependency (can start immediately) and any task on the
critical path (delays it = delays everything).

**Critical path:** [list task IDs in sequence that form the longest dependency chain]

---

### Risk register

Risks identified from PRD risk table, story NFRs, and integration points.

| Risk ID | Description | Likelihood | Impact | Mitigation | Owner | Trigger to escalate |
|---|---|---|---|---|---|---|
| R1 | [e.g., CIBIL API 9005 errors blocking loan origination] | High | High | Retry with exponential backoff + dead-letter queue | | If error rate > 5% in staging |
| R2 | | | | | | |

Likelihood and impact: High / Medium / Low.
Every HIGH×HIGH risk must have an owner and a mitigation before plan is approved.

---

### NFR implementation checklist

Maps PRD non-functional requirements to concrete implementation commitments.

| NFR dimension | Requirement (from PRD) | Implementation plan | Validation method | Phase |
|---|---|---|---|---|
| Performance | [e.g., p95 < 300ms on loan status API] | [e.g., DB index on loan_id + status] | Load test with k6 | Phase 2 |
| Security | [e.g., PAN masked in all logs] | [e.g., custom log serialiser] | Log audit in staging | Phase 1 |
| Scalability | | | | |
| Reliability | [e.g., 99.9% uptime on LOS APIs] | [e.g., retry + circuit breaker] | Chaos test | Phase 3 |

---

### Environment and deployment plan

| Step | Action | Environment | Validation | Rollback trigger |
|---|---|---|---|---|
| 1 | DB migration | Staging | Query result check | Any row count mismatch |
| 2 | Deploy service | Staging | Smoke test all endpoints | Any 5xx on critical paths |
| 3 | Integration test | Staging | Full test suite green | Any test failure |
| 4 | DB migration | Production | Same as staging | Same as staging |
| 5 | Deploy service | Production | Smoke test | Any 5xx rate > 0.1% |
| 6 | Monitor | Production | Error rate, latency, queue depth | Exceeds SLA threshold |

**Feature flag strategy:** [if applicable — flag name, rollout %, kill switch owner]
**Zero-downtime requirement:** [yes/no — if yes, blue/green or rolling strategy]

---

### Open questions and blockers

| # | Question / blocker | Type | Owner | Resolution needed by | Status |
|---|---|---|---|---|---|
| 1 | [e.g., Ozonetel API rate limit confirmed?] | Integration | | Before Phase 2 starts | Open |

Every open question must have an owner and a deadline. Unresolved blockers prevent
plan approval.

---

### Approval checklist

This plan is ready for the approval gate when all items below are checked:

- [ ] All PRD requirements are covered by at least one task
- [ ] All story ACs are covered by at least one task
- [ ] No scope has been added beyond PRD + stories
- [ ] All HIGH×HIGH risks have mitigations and owners
- [ ] All open questions have owners and deadlines
- [ ] Critical path is identified
- [ ] NFR requirements are mapped to implementation tasks
- [ ] Security requirements are mapped to Phase 1 tasks
- [ ] DB migration plan exists for all schema changes
- [ ] Deployment plan exists with rollback triggers

**Approval gate instruction (output this verbatim after plan is generated):**

---

> **This plan is a draft. Review every section before approving.**
>
> To edit: tell me which section to revise and what to change.
> To approve: reply **"Plan approved — proceed to execution"**
>
> Execution begins only after explicit approval.
> After approval, run `/secure-coding-practice` first, then `/red-green-tdd` inside that scaffold.

---

## Review mode

When an existing plan is provided for review, produce a gap report:

| Section | Status | Gap / issue |
|---|---|---|
| Header block | ✅ / ⚠️ / ❌ | |
| Scope confirmation | ✅ / ⚠️ / ❌ | |
| Architecture overview | ✅ / ⚠️ / ❌ | |
| Implementation phases | ✅ / ⚠️ / ❌ | |
| Cross-cutting concerns | ✅ / ⚠️ / ❌ | |
| Dependency map | ✅ / ⚠️ / ❌ | |
| Risk register | ✅ / ⚠️ / ❌ | |
| NFR checklist | ✅ / ⚠️ / ❌ | |
| Deployment plan | ✅ / ⚠️ / ❌ | |
| Open questions | ✅ / ⚠️ / ❌ | |
| Approval checklist | ✅ / ⚠️ / ❌ | |

Legend: ✅ Complete | ⚠️ Partial — needs expansion | ❌ Missing entirely

After gap report: "I can fill the missing sections now — provide the needed inputs,
or I will mark them as insufficient."

---

## Confidence score

Append after every generated plan:

```
Plan confidence: [0–100]%
Basis: [one sentence — what's solid vs. uncertain]
```

Below 80%: list which sections carry low confidence and what input would resolve it.