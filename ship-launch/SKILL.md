---
name: ship-launch
description: >
  Pre-launch verification and staged rollout. Use this skill whenever work is about to reach
  real users — a production deploy, a feature-flag flip, a release, a launch, a migration
  going live. Also trigger proactively when a task ends with "and then we ship it": the
  checklist and rollout plan below are the gate. This is the release twin of /preflight —
  preflight validates the environment before work begins; ship-launch validates the work
  before users receive it. The core stance: deploys are only safe when they are boring,
  observable, and reversible.
---

# Shipping & Launch

You are getting a change to users without an incident. Three properties make that possible,
and this skill enforces all three: the change is **verified** before exposure, **observable**
during exposure, and **reversible** at every stage.

---

## Step 1 — Pre-Launch Checklist

Run every section; report each item as pass / fail / not-applicable-because. Silent skips
are how incidents start.

**Correctness**
- Full test suite green; the feature verified end-to-end in a production-like environment,
  not just unit-tested.
- Code reviewed against the spec (`/pr-review`); no debug statements, no TODO-before-launch
  left, no commented-out safety checks.

**Security**
- No secrets in code, config, or logs; authn/authz verified on every new surface; input
  validation at the boundary (`/secure-coding-practices` for the full posture).

**Performance**
- Within budget under realistic data volume (`/perf-optimization` if a budget exists);
  new queries indexed; payloads sane.

**Operability**
- Telemetry in place for the new surface — the 3am test from `/observability`.
- Health checks cover the new dependency; runbook or at least a "what to check first" note
  exists for on-call.

**Data & Infra**
- Migrations tested against production-shaped data, and **backwards-compatible** with the
  currently running code (expand → migrate → contract; never a migration that breaks the
  old version while it's still serving).
- Config/env vars present in every target environment; DNS/TLS/quotas confirmed for anything
  new.

**Communication**
- Changelog/release notes written for the consumer; anyone affected (support, dependent
  teams) knows it's coming.

---

## Step 2 — Rollback Plan (before, not during)

Write it down before the deploy:
- **Mechanism**: flag off, revert commit, redeploy previous version — which one, and who can
  execute it.
- **Data reversibility**: if the change writes new data shapes, how does the old code handle
  them? If it can't, the rollback plan is incomplete — fix that first.
- **Triggers**: the specific numbers that mean roll back (see Step 3).
- Rollback must be a routine, pre-tested action, not an emergency invention.

---

## Step 3 — Staged Rollout

Decouple deploy from release: ship the code dark (flag off), then widen exposure in stages.

1. **Staging** — full verification in a production-like environment.
2. **Production, dark** — deployed, flag off, system healthy.
3. **Internal** — team/dogfood exposure; soak long enough to catch the obvious.
4. **Canary** — small user slice (~5%); watch error rate, latency, and the feature's own
   domain metric against baseline.
5. **Widen** — 25% → 50% → 100%, checking the same numbers at each step.
6. **Clean up** — after stable full rollout, remove the flag and dead path
   (a `/code-simplify` slice), close the cycle with `/context-sync`.

**Advance/hold/rollback rule:** metrics at baseline → advance; mildly elevated → hold and
investigate; error rate ~2× baseline, latency +50%, or any data-integrity or security signal
→ roll back immediately, diagnose offline (`/debug-triage`). Rolling back is the cheap,
professional move — never debug forward on users.

Scale the ceremony to the blast radius: a copy tweak doesn't need a canary; anything touching
money, auth, data shape, or send-to-humans (emails, campaigns) needs every stage. This applies
to GTM launches too — a `/marketing-automation-builder` journey going live is a deploy with
human recipients.

---

## Step 4 — Post-Launch Verification

- Actively verify in production: exercise the flow as a real user, watch dashboards through
  the first meaningful traffic — don't just wait for alerts.
- Compare the domain metric to the launch's stated objective (the spec or brief's KPI).
- Timing discipline: launch when people are around to respond — not at 6pm Friday.

---

## Exit Criteria

1. Checklist complete, every item pass or explicitly N/A.
2. Rollback plan written, mechanism tested, triggers numeric.
3. Rollout staged with metrics checked at each gate — or blast radius explicitly judged small
   enough to skip stages.
4. Production behaviour verified by observation; flag debt scheduled for cleanup.
