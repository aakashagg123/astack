---
name: incremental-implementation
description: >
  Thin vertical slices instead of big-bang builds. Use this skill whenever an implementation
  will span multiple files, exceed ~100 lines before anything can be tested, or execute a
  multi-story plan from /tech-development-plan. Also trigger proactively when you catch
  yourself building horizontally — all the models, then all the services, then all the UI —
  because horizontal layers can't be verified until everything meets at the end. Every
  increment must leave the system working, tested, and committable on its own.
---

# Incremental Implementation

You are building in thin vertical slices. A slice cuts through every layer of the stack to
deliver one narrow piece of working behaviour — testable and committable on its own. The
system is never more than one small increment away from a working state.

---

## Why Vertical Beats Horizontal

Horizontal building (all schemas → all services → all endpoints → all UI) defers every
integration risk to the very end, where it is most expensive to discover. A vertical slice
surfaces the integration problem in the *first* increment, when the fix costs minutes.

---

## Choosing the Slice Order

1. **Risk-first** — take the slice with the most uncertainty first (the unfamiliar API, the
   gnarly data model, the performance question). If it kills the approach, you want that
   news on day one, not day five. Pairs with `/evaluate-approach`.
2. **Walking skeleton** — the thinnest end-to-end path that touches every layer, even if it
   only handles one hard-coded case. Everything after is widening, not wiring.
3. **Contract-first** — when slices could parallelize, fix the interface between them first
   (see `/api-design`), then fill in both sides independently.

---

## The Increment Cycle — every slice, no exceptions

1. **Scope** — state the slice in one sentence of user-visible behaviour ("a user can X for
   the simplest case Y").
2. **Implement** — smallest complete version. Prefer the naive-but-clear implementation;
   abstraction gets earned by the *second* slice that needs it, never anticipated by the first.
3. **Test** — run the suite; write the slice's tests (via `/red-green-tdd` when doing TDD).
4. **Verify working state** — build passes, types/lint clean, the behaviour actually runs.
5. **Commit** — one logical change, descriptive message, independently revertable.
6. **Next slice.**

If any step fails, fix it before starting the next slice — never stack an increment on a
broken state (`/debug-triage`'s stop-the-line rule applies here too).

---

## Incomplete Work Stays Invisible

- Gate unfinished behaviour behind a feature flag or an unexposed entry point, so every
  commit is deployable even mid-feature.
- Default the flag to off; defaults must be safe.
- When the feature fully ships, removing the flag is its own final slice (dead flags are
  `/code-simplify` fodder).

---

## Scope Discipline

- **One logical change per increment.** Refactoring that a slice *requires* goes in its own
  preparatory commit before the slice, not blended into it.
- **No adjacent cleanup.** Noticed messy code nearby? Note it for a later `/code-simplify`
  pass. Touching unrelated files mid-slice destroys the diff's reviewability and the
  commit's revertability.
- **No mid-slice scope growth.** New ideas go on the slice list, not into the current diff.

---

## Anti-Patterns to Refuse

- "I'll test everything once it's all wired up" — that's a big bang with extra steps.
- "It doesn't compile between commits, but the final one fixes it" — every commit must build.
- "I'll add the abstraction now since slice 4 will need it" — slice 4 gets to decide that.
- "Quick unrelated fix while I'm in this file" — separate commit, separate intent.

---

## Exit Criteria (per slice)

1. Slice statable in one sentence and delivered end to end.
2. Suite green, build clean, behaviour verified running.
3. Committed atomically; reverting this commit removes exactly this slice.
4. Incomplete surface area flagged off or unexposed.
