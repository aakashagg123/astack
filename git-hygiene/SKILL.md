---
name: git-hygiene
description: >
  Git discipline — branching, commit hygiene, history, and versioning. Use this skill whenever
  the user asks about branching strategy, commit messages, rebasing vs merging, release
  tagging, semantic versioning, or untangling a messy history. Also trigger proactively when
  you are about to commit work: it defines what a good commit looks like in this stack.
  The goal is a history that functions as documentation — every commit a reviewable,
  revertable unit with a message that explains why.
---

# Git Hygiene & Versioning

You are keeping version history useful. Git history is the only documentation guaranteed to
exist for every change; treat each commit as a message to the engineer debugging this code at
2am two years from now (probably you).

---

## Branching

- **Short-lived branches off main, merged fast.** A branch is a workspace, not a home. The
  longer it lives, the worse the merge — integrate within days, not weeks.
- **One branch, one purpose.** Feature work, bug fix, and refactor each get their own branch.
- Incomplete features are handled with flags and unexposed entry points
  (see `/vertical-slicing`), not by parking branches for a month.
- Name branches by intent: `fix/login-timeout`, `feat/export-csv` — a reader should know the
  point without opening the diff.
- **Never rewrite shared history.** Rebase and amend freely on branches only you hold; the
  moment others may have pulled it, history is append-only. Force-push only with
  `--force-with-lease`, only on your own branches.

---

## Commits

- **Atomic.** One logical change per commit — it builds, tests pass, and reverting it removes
  exactly one thing. "WIP", "fixes", and "more changes" are not commits; squash them into
  their logical parent before merging.
- **Separate the noise.** Formatting-only, rename-only, and generated-file changes go in their
  own commits so the substantive diff stays reviewable.
- **Messages explain why.** Subject line: imperative, ≤ 72 chars, specific
  ("Debounce search input to cut API load", not "Update search"). Body: the why — the
  constraint, the bug, the trade-off — because the diff already shows the what. Reference the
  issue/spec where one exists.
- If the project uses a message convention (Conventional Commits, ticket prefixes), match it —
  check `git log` before inventing your own style.

---

## Merging & Review Flow

- Keep PRs slice-sized: a reviewable PR covers one concern and is small enough to review
  properly in one sitting. Big features arrive as a sequence of PRs, not one monolith
  (route review itself through `/pr-review`).
- Update your branch by rebasing onto main (or merging main in, per project convention) and
  resolve conflicts *on the branch*, so main's history stays linear-ish and green.
- Merge only green: build, tests, lint pass before merge — no "fix it on main after".

---

## Versioning & Releases

- **Semantic versioning** for anything with consumers: MAJOR = breaking, MINOR = additive,
  PATCH = fixes. A breaking change in a minor version is a broken promise, not a version.
- Tag releases (`v2.3.0`) so any production state can be reproduced exactly.
- Keep a changelog entry per release written for the *consumer*: what changed for them, what
  breaks, how to migrate. Coordinate the deprecation window with `/contract-design`'s evolution
  rules and the rollout with `/launch-gate`.

---

## Recovering Safely

- Undo a shipped commit with `git revert` (new commit, history intact) — never reset a shared
  branch.
- Before any destructive local operation (rebase, reset --hard), note the current SHA;
  `git reflog` can bring back almost anything, but only if you look before garbage collection.
- Locate regressions with `git bisect` — it's the payoff for keeping every commit buildable.

---

## Exit Criteria (per unit of work)

1. Branch contains only its stated purpose.
2. Every commit atomic, buildable, message explains why.
3. Noise (formatting/renames/generated) separated from substance.
4. Merged green; shared history never rewritten; release tagged and changelogged when a
   version shipped.
