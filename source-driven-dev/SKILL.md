---
name: source-driven-dev
description: >
  Ground implementation decisions in official documentation, not memory. Use this skill
  whenever writing framework- or library-specific code where the API surface changes across
  versions — configuration, integrations, SDK calls, build tooling — or when the user asks
  for "the current best practice" or "the right way" to do something. Also trigger proactively
  before generating any non-trivial usage of a dependency you haven't verified against the
  project's actual installed version: training-data memory ages, APIs deprecate, and a
  confidently written call to a renamed method is the most common form of hallucinated code.
---

# Source-Driven Development

You are writing code whose correctness depends on external APIs — so you verify against the
primary source instead of trusting recall. Memory of a library is a snapshot that started
aging the day it was taken; the library kept moving.

---

## When It Applies — and When It Doesn't

**Verify** when the code touches: framework configuration and lifecycle, library/SDK calls
with evolving signatures, build/deploy tooling, web platform features with compatibility
ranges, or any pattern that will be copied across the codebase (boilerplate compounds —
verify once, save N corrections).

**Skip** when the code is pure logic — algorithms, data transforms, project-internal
functions — where correctness doesn't depend on anyone else's API. Don't ritualise
verification where nothing external is being trusted; the user's explicit speed-over-
verification call also wins.

---

## The Four-Step Loop

### 1. Pin the actual versions

Read the dependency manifest and lockfile (`package.json`, `requirements.txt`,
`pyproject.toml`, `go.mod`, `Gemfile`…) before writing a line. The question is never "how
does library X work" — it's "how does library X **at version Y** work." If the version is
ambiguous or about to be chosen, surface that as a decision, not a guess.

### 2. Consult the primary source

Trust order, descending:
1. Official documentation for the pinned version (versioned docs beat `latest`).
2. Official changelogs, migration guides, and maintainer announcements.
3. Web standards and platform references (specs, MDN-class references) for platform features.
4. The library's own source/types in `node_modules`/site-packages — the ground truth when
   docs are thin.

Explicitly downrank: tutorials, Stack Overflow answers of unknown vintage, blog posts, and
your own recall — all snapshots of *some* version, rarely yours. Fetch the specific feature
page, not the docs homepage.

### 3. Implement to the documented contract

- Match the documented signatures, options, and recommended patterns for the pinned version;
  prefer the current recommended API over the legacy one you remember.
- When the docs and the existing codebase disagree, don't silently pick either — flag it:
  the codebase may be deliberately pinned, or overdue for migration.
- If something couldn't be verified (offline, undocumented corner), say so at the call site
  and in the summary — an explicit "unverified" beats confident fiction.

### 4. Leave a trail

For non-obvious choices, record the source next to the decision — a doc URL in the relevant
comment, spec, or PR description — so the next reader (or `/pr-review`) can re-verify in one
click instead of re-deriving the research.

---

## Signals You Skipped This Skill

- A method call that doesn't exist in the installed version (the classic).
- Deprecation warnings on freshly written code.
- Config options the tool no longer reads, failing silently.
- Hedging in prose ("this should work…") in place of checking.
- Modern-looking code using a pattern the framework retired two majors ago.

---

## Exit Criteria

1. Versions read from the manifest, not assumed.
2. Every external API usage traceable to a primary source for that version — or explicitly
   marked unverified.
3. Docs-vs-codebase conflicts surfaced, not silently resolved.
4. Sources cited where the choice wasn't obvious.
