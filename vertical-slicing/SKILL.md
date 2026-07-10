---
name: vertical-slicing
description: >
  Building in thin vertical slices instead of big-bang layers. Use this skill whenever an
  implementation spans multiple files, would exceed ~100 lines before anything can run, or
  executes a multi-story plan from /tech-development-plan. Also trigger proactively when you
  catch yourself building horizontally — all the models, then all the services, then all the
  UI — because horizontal layers can't prove anything until they finally meet, and by then
  the integration surprises are maximally expensive. The invariant this skill protects:
  the system is never more than one small, committable step away from working.
---

# Vertical Slicing

You are cutting a feature into slices that each run end to end. A slice goes *through* the
stack — a sliver of UI to a sliver of logic to a sliver of storage — and delivers one narrow
piece of real behaviour that can be tested, committed, and reverted on its own.

The reason is risk timing. Layered building answers "does it all fit together?" last, at the
most expensive possible moment. The first vertical slice answers it first, when the fix
costs minutes. Everything after the first slice is widening, not wiring.

---

## Cutting the First Slice

Three cuts, chosen by what the feature most needs to prove:

- **The skeleton cut** — the thinnest path that touches every layer, even for one hard-coded
  case. Right default when integration itself is the unknown.
- **The risk cut** — start where uncertainty is highest: the unfamiliar API, the gnarly data
  model, the performance question. If the approach is going to die, learn it on day one
  (pairs with `/evaluate-approach`).
- **The seam cut** — when work could parallelize, fix the interface between the parts first
  (`/contract-design`), then fill both sides independently against the agreed seam.

---

## The Slice Contract

Every slice, without exception, satisfies four clauses before the next slice begins:

- **Statable** — one sentence of user-visible behaviour: "a user can X for the simplest
  case Y." Can't state it? It's a layer, not a slice — re-cut.
- **Naive by default** — the clear, direct implementation. An abstraction is *earned* by the
  second slice that needs it; the first slice never pre-builds one on speculation.
- **Proven** — its tests written and green (through `/red-green-tdd` when running TDD),
  build passing, the behaviour actually exercised.
- **Committed whole** — one logical change, descriptive message; reverting the commit
  removes exactly this slice and nothing else.

A failed clause halts the line — the broken-state rule from `/root-cause` applies to
half-finished slices too. And commit hygiene across slices is `/git-hygiene`'s territory:
this skill decides what a commit contains; that one decides how it reads.

---

## Dark Until Done

Mid-feature, the trunk stays shippable:

- Unfinished behaviour hides behind a feature flag or an unwired entry point — deployable
  is a property of every commit, not of the final one.
- Flags default off; a default must be the safe state.
- The flag's removal, once the feature fully lands, is itself the final slice — a lingering
  dead flag is `/code-declutter` work someone else inherits.

---

## Scope Walls

The walls that keep a slice thin:

- **Refactor-before, never refactor-during.** Restructuring a slice genuinely needs goes in
  its own preparatory commit ahead of the slice — never blended in.
- **The sighting log.** Messy code spotted nearby gets *written down* (for a
  `/code-declutter` pass), not fixed en passant. Adjacent cleanup destroys both the diff's
  reviewability and the commit's revertability.
- **New ideas join the queue.** Mid-slice inspiration goes on the slice list, not into the
  working diff.

The tell that a wall is breached: the diff touches a file the slice sentence doesn't explain.

---

## Refusals

- "I'll test it all once it's wired up" — a big bang wearing increments as a costume.
- "The build breaks between these commits, the last one fixes it" — every commit builds.
- "Slice four will need this abstraction, adding it now" — slice four gets that vote.
- "Tiny unrelated fix while I'm in the file" — separate commit, separate intent.

---

## Exit Criteria (per slice)

1. Slice sentence written; delivered end to end.
2. All four contract clauses met — proven, naive-by-default, committed whole.
3. Unfinished surface dark: flagged off or unwired.
4. Sighting log captured; zero adjacent cleanup smuggled into the diff.
