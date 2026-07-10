---
name: docs-grounding
description: >
  Grounding external-API usage in current official documentation instead of memory. Use this
  skill whenever writing framework- or library-specific code where the surface changes across
  versions — configuration, integrations, SDK calls, build tooling — or when the user asks
  for "the current best practice" or "the right way" to do something. Also trigger proactively
  before generating any non-trivial usage of a dependency you haven't checked against the
  project's installed version: recall of a library is a snapshot that started aging the day
  it was taken, and a confidently written call to a renamed method is the most common form of
  hallucinated code.
---

# Docs Grounding

You are writing code whose correctness depends on someone else's API — so every claim about
that API gets a source better than your own recall. The question is never "how does library
X work"; it is always "how does library X **at the version this project installs** work",
and the manifest, not memory, answers the first half.

---

## Scope — where grounding is owed

**Owed** wherever an external contract is being trusted: framework configuration and
lifecycle, library and SDK calls with evolving signatures, build and deploy tooling, web
platform features with compatibility ranges — and *especially* any pattern about to be
replicated across the codebase, because boilerplate compounds: verify once or correct it N
times.

**Not owed** for pure logic — algorithms, data transforms, project-internal functions —
where nothing external is being trusted. Grounding is a discipline, not a ritual; and an
explicit speed-over-verification call from the user always wins.

---

## The Grounding Rule

> No external API call ships unless its signature and semantics are traceable to a primary
> source for the pinned version — or it is explicitly labelled unverified.

Everything below serves that rule.

**Pin first.** Read the dependency manifest and lockfile (`package.json`,
`requirements.txt`, `pyproject.toml`, `go.mod`, `Gemfile`…) before writing a line. If the
version is ambiguous — or the task *is* choosing it — that's a surfaced decision, not a
silent default.

**Then climb no higher on the trust ladder than you must:**

| Rung | Source | Trust it for |
|---|---|---|
| 1 | Versioned official docs, changelogs, migration guides | Everything |
| 2 | Web standards and platform references (specs, MDN-class) | Platform features, compatibility |
| 3 | The dependency's own source and type definitions on disk | Ground truth when docs are thin |
| 4 | Tutorials, Q&A threads, blog posts | Leads to verify on rungs 1–3, never answers |
| 5 | Your own recall | Hypotheses only |

Fetch the specific feature page for the pinned version, not the docs homepage; `latest`
docs describe someone else's install.

---

## Reconciliation — when sources disagree

- **Docs vs. codebase**: the existing code may be deliberately pinned to an older pattern,
  or overdue for migration — either way it's flagged, never silently overruled in one file
  (that inconsistency is its own bug).
- **Docs vs. recall**: docs win, and the surprise is worth noting — it usually means a
  deprecation your memory predates.
- **Unverifiable** (offline, undocumented corner): implement with an explicit *unverified*
  label at the call site and in the summary. A labelled guess is honest; a confident one is
  fiction.

---

## Traceability

Non-obvious choices carry their receipt: a doc URL beside the decision — in the comment,
the spec, or the PR description — so `/pr-review` (or you, in six months) can re-verify in
one click instead of re-deriving the research. This pairs with `/context-curation`: a cited
source in context beats a remembered one in your head.

---

## Symptoms of Ungrounded Code

- A method call that doesn't exist in the installed version — the classic.
- Deprecation warnings on freshly written lines.
- Config keys the tool silently no longer reads.
- Hedging prose ("this should work…") standing in for a check.
- A pattern the framework retired two major versions ago, written with total confidence.

---

## Exit Criteria

1. Versions read from the manifest, never assumed.
2. Every external call traceable to rungs 1–3 for that version, or labelled unverified.
3. Disagreements between docs, codebase, and recall surfaced, not silently resolved.
4. Receipts attached where the choice wasn't obvious.
