---
name: code-declutter
description: >
  Complexity reduction with behaviour preserved. Use this skill whenever the user asks to
  simplify, clean up, refactor for readability, remove dead code, reduce duplication, or
  "make this less messy". Also trigger proactively after a feature lands — when you notice
  needless abstraction layers, copy-paste blocks, unused exports, or a function no reader could
  summarise in one sentence. This is quality work, not bug hunting: the contract is that
  observable behaviour is identical before and after. If tests don't exist to prove that,
  creating the behaviour lock is step one, not optional.
---

# Code Declutter

You are reducing complexity while provably preserving behaviour. Simplification that changes
behaviour is not simplification — it's an unreviewed feature change wearing a cleanup label.

---

## Step 0 — Behaviour Lock

Before touching anything, establish proof that behaviour is preserved:
- Identify the tests covering the code you'll change. Run them; record green.
- If coverage is missing, write characterization tests first — tests that pin down what the
  code *currently does* (including its quirks), not what it should do.
- No lock, no simplification. This is the skill's only hard gate.

---

## Step 1 — Find the Complexity Worth Removing

Scan for these, in order of payoff:

1. **Dead code** — unused exports, unreachable branches, commented-out blocks, feature flags
   fully rolled out. Deletion is the highest-value refactor: zero risk to readers, negative
   maintenance cost.
2. **Needless abstraction** — interfaces with one implementation, wrappers that only forward,
   config for things that never vary, generic machinery serving one call site. Collapse them.
   The rule: abstraction must be *earned* by a second real use, not anticipated.
3. **Duplication with divergence risk** — the same logic in 2+ places that must change
   together. Extract only what genuinely repeats; near-duplicates that differ on purpose
   should stay separate.
4. **Deep conditionals** — nesting beyond 2–3 levels. Prefer early returns, guard clauses,
   and lookup tables over else-chains.
5. **Oversized units** — functions doing several jobs, files with several reasons to change.
   Split along responsibility, not line count.
6. **Dependency weight** — a library imported for one function you could write in ten lines,
   or duplicated by the platform now.

---

## Step 2 — Simplify in Reviewable Increments

- One category of change per commit. Never mix "delete dead code" with "restructure module"
  with "rename for clarity" in a single diff.
- Run the behaviour lock after every increment, not just at the end.
- Pure moves/renames go in their own commit so the interesting diff stays readable.
- Match the codebase's existing idiom. A "simplification" that introduces a foreign style adds
  the very cognitive load it claims to remove.

---

## What NOT to Do

- **Don't simplify hot paths blind.** If code looks contorted for performance, check for a
  benchmark or comment first; measure before and after if you touch it (see `/perf-tuning`).
- **Don't chase metrics.** Cyclomatic-complexity scores and line counts are hints, not goals.
  Optimise for the next reader's comprehension time.
- **Don't "improve" behaviour.** Found a probable bug while simplifying? Record it, finish the
  behaviour-preserving pass, then fix the bug separately through `/root-cause`. One diff, one
  intent.
- **Don't add cleverness.** A dense one-liner replacing five clear lines is compression, not
  simplification.

---

## Exit Criteria

1. Behaviour lock green before and after every increment.
2. Net complexity down: fewer indirections, fewer lines a reader must hold in their head — and
   you can say *what* got simpler in one sentence per commit.
3. No behavioural diffs smuggled in (review your own diff for logic changes before finishing).
4. Public API unchanged, or changes explicitly called out and approved.
