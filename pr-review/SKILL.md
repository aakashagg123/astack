---
name: pr-review
description: >
  Structured code review against the originating PRD, user stories, and acceptance criteria.
  Validates that generated or submitted code actually delivers what was specified — not just
  that it runs. Use this skill when a developer or PM wants to review a code diff, PR, or
  implementation against requirements. Trigger on: "review this PR", "review this code",
  "does this code match the PRD", "validate implementation", "run pr-review", "check my
  code against stories", "is this shippable", "code review", "review before merge", or any
  request to assess code quality, correctness, or requirements traceability. Also trigger
  proactively when a conversation transitions from code generation to validation. This skill
  is requirements-first — it reviews code against spec, not just against style guides.
  Always requires at minimum the code and one of: PRD, user stories, or acceptance criteria.
---

# PR Review — Skill Guide

Validates that code delivers what was specified. Two failure modes this skill prevents:
(1) code that works but doesn't match the spec, and (2) code that matches the spec but
ships insecure, untested, or unmaintainable. Both are production incidents waiting to happen.

---

## Behaviour rules

**Rule 1 — Spec-first, always.**
Every code review starts from the requirements artifacts (PRD, stories, ACs). If none are
provided, ask for them before reviewing. Style and quality feedback without requirements
traceability is incomplete.

**Rule 2 — Structured output, no free-form essays.**
Output follows the fixed report format below. No narrative reviews. Verdicts are explicit:
PASS / FAIL / WARN per dimension.

**Rule 3 — Evidence-linked findings.**
Every FAIL or WARN must cite: (a) the specific requirement or AC it violates, and
(b) the specific code location (file, function, line if available).

**Rule 4 — Severity tagging is mandatory.**
Every finding is tagged: `[BLOCKER]`, `[MAJOR]`, or `[MINOR]`.
- BLOCKER: ships with this = production incident or requirements breach
- MAJOR: ships with this = tech debt, security gap, or likely bug
- MINOR: ships with this = maintainability or style degradation

**Rule 5 — Shippability verdict is binary.**
The final verdict is SHIP or HOLD. No "ship with caveats" — if there's a BLOCKER, it's HOLD.

**Rule 6 — Don't rewrite code in the review.**
Flag findings with precise descriptions. Optionally provide a one-line fix hint. Full
rewrites belong in a separate code generation step, not in the review output.

---

## Intake protocol

Ask all questions in one batch if any are missing from context:

1. What is the feature / PR being reviewed? (name or Jira ref)
2. What artifacts are available? (PRD, user stories, ACs, tech plan — paste or summarise)
3. What is the code diff or implementation? (paste inline or describe modules)
4. What stack / framework is this in?
5. Was `/secure-coding-practice` applied during generation? (Y/N — affects security
   review depth)
6. Was `/red-green-tdd` applied? (Y/N — affects test coverage expectations)
7. Are there known constraints to check against? (performance SLAs, regulatory, data
   residency, API rate limits)
8. Is this a net-new feature, an extension, or a fix?

---

## Review dimensions

Run all dimensions in sequence. Each produces a PASS / FAIL / WARN verdict.

### 1. Requirements traceability
Every user story and AC in scope should map to at least one testable code path.
- Are all in-scope stories addressed?
- Are any out-of-scope items implemented (scope creep)?
- Are edge cases from ACs handled explicitly?

### 2. Functional correctness
Does the code do what the spec describes?
- Happy path matches expected behaviour?
- Error paths and edge cases handled?
- Data contracts (inputs/outputs) match what PRD specifies?

### 3. Test coverage
- Are unit tests present for all non-trivial logic?
- Do test names map to AC language? (traceability)
- Are tests testing behaviour, not implementation?
- Are failure cases tested, not just happy paths?
- If TDD was applied: is there evidence of Red-Green-Refactor discipline?

### 4. Security posture
- Input validation present on all external inputs?
- No sensitive data in logs, URLs, or error messages?
- Auth/authz checks on all protected paths?
- No hardcoded secrets, tokens, or credentials?
- SQL / NoSQL injection vectors handled?

### 5. Code quality and maintainability
- Functions are single-responsibility?
- No deeply nested conditionals without clear justification?
- No dead code or commented-out blocks?
- Naming is unambiguous and domain-consistent?
- No magic numbers or strings — constants named and placed?

### 6. Integration and dependency risk
- External API calls have timeout and retry handling?
- Third-party failures are graceful (no unhandled exceptions propagating to UI)?
- Database queries have appropriate indexing considerations flagged?

### 7. Observability
- Key operations logged at appropriate levels?
- Errors are logged with enough context to debug in production?
- No PII in log statements?
- Critical flows have traceability (correlation IDs, request IDs)?

---

## Output format

### PR Review Report — [Feature Name] — [Date]

**Artifacts reviewed:** [list]
**Stack:** [framework / language]
**TDD applied:** Y/N | **Secure coding applied:** Y/N

---

**Dimension verdicts:**

| Dimension | Verdict | Findings |
|---|---|---|
| Requirements traceability | PASS / FAIL / WARN | N findings |
| Functional correctness | PASS / FAIL / WARN | N findings |
| Test coverage | PASS / FAIL / WARN | N findings |
| Security posture | PASS / FAIL / WARN | N findings |
| Code quality | PASS / FAIL / WARN | N findings |
| Integration risk | PASS / FAIL / WARN | N findings |
| Observability | PASS / FAIL / WARN | N findings |

---

**Findings:**

```
[SEVERITY] Dimension: <dimension name>
Requirement: <story / AC / PRD section violated>
Location: <file / function / line>
Issue: <precise description of what's wrong>
Fix hint: <one-line direction, not full rewrite>
```

_(Repeat for each finding, ordered: BLOCKER → MAJOR → MINOR)_

---

**Requirements coverage:**

| Story / AC | Status | Notes |
|---|---|---|
| [Story ID] [AC text] | ✅ Covered / ❌ Missing / ⚠️ Partial | |

---

**Final verdict: SHIP / HOLD**

`Reason: [one sentence. If HOLD: which BLOCKER(s) must be resolved first.]`

---

**Suggested next action:**
- If SHIP: flag any MINOR items for backlog, propose `/context-sync` to capture learnings.
- If HOLD: list exact BLOCKERs with owners and resubmit instructions.

---


## Confidence score

Close every report with:
`Review confidence: X% — [N] dimensions fully reviewed. [Gaps if any artifacts were missing.]`

Below 75%: state which dimension was limited by missing artifacts and what would improve coverage.