---
name: red-green-tdd
description: >
  Enforces strict Red/Green Test-Driven Development discipline for any coding task. Use this skill
  whenever a user mentions TDD, test-driven development, red/green TDD, test-first, or asks to
  write code "with tests". Also trigger proactively when a user asks Claude to implement a feature,
  fix a bug, or build a module — because TDD is the right default for coding agents. This skill
  prevents two of the most common coding agent failure modes: writing code that doesn't work, and
  writing code that never gets called. It enforces the write-tests-first → confirm-failure →
  implement → confirm-pass cycle. If the user says "use red/green TDD" or "TDD this", treat it as
  a full workflow invocation, not a hint.
---

# Red/Green TDD Skill

## What this skill does

Runs a strict test-first development loop for any coding task. Every piece of code produced must be:
1. Preceded by a failing test (Red)
2. Followed by an implementation that makes it pass (Green)

No exceptions. No "I'll add tests later." That's not TDD — that's wishful thinking.

---

## Why TDD is especially powerful for coding agents

Coding agents have two signature failure modes:

- **Hallucinated correctness**: code that looks plausible but doesn't work
- **Dead code**: implementation that exists but is never exercised

Test-first development kills both. A failing test proves the feature was missing. A passing test proves the implementation actually runs. The test suite becomes the proof of work.

---

## The Red/Green Cycle — enforced, not suggested

### Phase 1: RED

1. **Understand the requirement** — one unit of behaviour at a time
2. **Write the test first** — before any implementation code exists
3. **Run the test** — confirm it fails with the *right* failure
   - A compile/import error means setup is broken — fix setup first
   - An assertion failure means the test is correctly detecting absence of the feature — ✅
   - A passing test at this stage means either the feature already exists (no work needed) or the test is wrong (fix the test)
4. **Do not proceed** to Phase 2 until the test fails for the right reason

> The red phase is not a formality. Skipping it is how you build a test suite full of tests that prove nothing.

### Phase 2: GREEN

5. **Write the minimum implementation** to make the test pass — no more.
   "Minimum" means: make this one test go green. Use the simplest code that achieves it:
   - If a hardcoded return value passes the test — that's the minimum (for now)
   - If real logic is needed — write the simplest version of that logic only
   - If a complex feature has multiple behaviours, each behaviour gets its own cycle
   Do not implement the next feature because it seems obvious. Write the next test first.
6. **Run the test** — confirm it passes
7. **Run the full suite** — confirm nothing regressed

### Phase 3: REFACTOR (optional but encouraged)

8. Clean up the implementation if needed
9. Re-run the full suite — still green? Ship it.

---

## Scope and granularity

- One failing test → one implementation chunk → repeat
- Don't batch multiple features into one cycle
- Each cycle should take minutes, not hours
- If a cycle feels big, decompose it

---

## Test quality rules

A test that always passes is not a test. A test that tests the wrong thing is worse than no test.

**Good tests:**
- Test one behaviour
- Have a clear, descriptive name (`test_loan_amount_cannot_be_negative`, not `test_loan`)
- Assert the outcome, not the implementation detail
- Are fast and deterministic

**Anti-patterns to reject:**
- Tests written after the implementation ("retrofitted tests")
- Tests that mock everything and assert nothing real
- Tests with no assertion
- Tests that pass on the first run before any implementation exists (broken red phase)

---

## Language-agnostic — framework selection

Use whatever test framework is idiomatic for the project:

| Language | Default framework |
|----------|------------------|
| Python | `pytest` |
| JavaScript/TypeScript | `jest` or `vitest` |
| Java | `JUnit 5` |
| Go | `testing` (stdlib) |
| Rust | built-in `#[test]` |
| Ruby | `RSpec` |

If the project already has a framework, use it. Don't introduce a second one.

---

## Output format for each TDD cycle

When applying this skill, structure every response as:

```
## Cycle N: <behaviour being tested>

### RED — Test
<test code>

### Run the test (confirm RED)
<show the run command AND the expected failure output — e.g. "AssertionError", "undefined function",
"FAIL: expected X got nil". If actually running code: paste real output. If describing: state exactly
what error message proves the test is correctly detecting the missing feature. DO NOT skip this step.>

### GREEN — Implementation
<implementation code>

### Run the suite (confirm GREEN)
<show or describe: (1) the new test now passes, (2) the full suite is still green. For new projects
with no prior suite, state "no prior suite — this is test #1" and confirm the new test passes.>
```

---

## Integration with existing codebases

- Always read existing tests before writing new ones — match style and patterns
- Run the full suite at the start to establish a baseline (all tests should pass before you touch anything)
- If the baseline is already broken, surface it immediately — don't add more tests on top of a broken foundation

---

## What NOT to do

- ❌ Write implementation first, then "add tests to cover it"
- ❌ Skip the red phase because "obviously this will fail"
- ❌ Write multiple tests before any implementation
- ❌ Use TDD as a label but execute waterfall
- ❌ Mark a cycle complete without running the tests

---

## Bug-fix TDD variant

Bug fixes follow the same red/green discipline — just with a different starting point.

**The bug-fix cycle:**

1. **Write a test that reproduces the bug** — before touching any production code
   - The test must fail *because of the bug*, not for any other reason
   - If the test passes, either the bug is already fixed or your test isn't hitting the right code path
2. **Confirm the test fails** with the bug-induced failure (not a compile error, not wrong assertion)
3. **Fix the bug** — minimum change only
4. **Confirm the test now passes**
5. **Run the full suite** — the bug fix must not break anything else

> A bug without a failing test is a bug you'll see again. The test is the permanent guard.

**Example trigger:** "fix this bug / the function returns X when it should return Y"
→ Do NOT touch the function first. Write `test_function_returns_Y_when_[condition]` first.

---

## Shorthand recognition

If the user says any of the following, invoke this full workflow:
- `"use red/green TDD"`
- `"TDD this"`
- `"test-first"`
- `"write tests first"`
- `"red green"`

These are not style preferences. They are workflow invocations. Honour them fully.