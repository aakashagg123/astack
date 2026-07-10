---
name: launch-gate
description: >
  The gate between finished work and real users. Use this skill whenever anything is about to
  go live — a production deploy, a feature-flag flip, a release, a data migration, a campaign
  send. Also trigger proactively when a task ends with "and then we ship it": nothing crosses
  to users without passing this gate. It is the release twin of /preflight — preflight proves
  the environment is ready before work starts; launch-gate proves the work is ready before
  users receive it. The stance throughout: a deploy should be the most boring moment of the
  project, because everything risky was made boring in advance.
---

# Launch Gate

You are standing between a change and its users. The change passes only by demonstrating
three properties — **proven** before exposure, **watched** during exposure, **retractable**
at any point. Excitement at deploy time is a process failure; this gate is where the
excitement gets removed.

---

## Gate 1 — Readiness

Walk the change through each lens. Every item resolves to pass, fail, or
*not-applicable-because* — the silent skip is how incidents are born.

- **Behaviour** — suite green; the feature exercised end-to-end in a production-like
  environment; reviewed against its spec (`/pr-review`); no leftover debug output,
  disabled checks, or launch-blocking TODOs.
- **Security** — no secrets anywhere in code, config, or logs; authn/authz exercised on
  every new surface; boundary input validated (`/secure-coding-practices` owns the full
  posture).
- **Load** — realistic data volume tried; new queries indexed; budgets honoured
  (`/perf-tuning` where one exists).
- **Diagnosability** — the 3am test from `/telemetry-design`: could on-call diagnose this
  change's failure modes from telemetry alone? Health checks cover new dependencies; a
  "what to check first" note exists.
- **Data** — migrations rehearsed on production-shaped data and compatible with the code
  version still running during rollout (expand → migrate → contract; never a step that
  strands the currently deployed version).
- **People** — changelog written for the consumer; support and dependent teams not
  surprised.

---

## Gate 2 — Retractability

The rollback plan is written **before** the deploy, and it is boring on purpose:

- **The lever**: flag off / revert / redeploy previous — which one, who pulls it, how long
  it takes. If it has never been rehearsed, rehearse it now.
- **The data question**: once new-shape data exists, can the old code still run? If not,
  the change is not yet retractable — fix that before proceeding, not after.
- **The tripwires**: written thresholds, chosen per launch and agreed in advance — error
  rate, latency, and the feature's own domain metric relative to its pre-launch baseline.
  Deciding thresholds during an incident is how "let's watch it a bit longer" ships a
  bad night.

---

## Gate 3 — Staged Exposure

Deploying and releasing are two events. Ship the code dark, then widen exposure one
audience at a time, watching the tripwire metrics at every widening:

1. **Nobody** — deployed to production, flag off, system healthy.
2. **Us** — the team dogfoods; soak long enough to catch the obvious.
3. **A real slice** — a small share of genuine users; size it to the blast radius, not to
   a ritual percentage.
4. **Everybody** — widen stepwise while the metrics hold; each widening is a decision, not
   a default.
5. **Cleanup** — once stable at full exposure, remove the flag and dead path (a
   `/code-declutter` slice) and close the cycle with `/context-sync`.

At every step the rule is the same: metrics at baseline → widen; drifting → hold and
investigate; a tripwire crossed, or *any* data-integrity or security signal → pull the
lever immediately and diagnose offline with `/root-cause`. Rolling back is the cheap,
professional move — debugging forward on live users is neither.

**Proportionality.** A copy tweak doesn't need five stages; anything touching money, auth,
stored data shapes, or messages sent to humans needs all of them. That last category makes
this gate apply to GTM work too — a `/marketing-automation-builder` journey going live is a
deploy whose users have inboxes.

---

## Gate 4 — Confirmation

Passing the gate ends with observation, not assumption:

- Exercise the flow in production as a real user would.
- Watch the dashboards through the first meaningful traffic — actively, not
  waiting-for-alerts.
- Compare the domain metric against the objective the spec or brief promised (the KPI from
  `/spec-creator` or `/marketing-brief`).
- Launch when responders are awake. Nothing non-urgent goes out at the end of a Friday.

---

## Exit Criteria

1. Readiness walked: every item pass or explicitly N/A.
2. Rollback lever named, rehearsed, and tripwires written down before exposure began.
3. Exposure widened stage by stage with metrics checked — or blast radius explicitly
   judged small enough to collapse stages.
4. Production behaviour confirmed by observation; flag debt scheduled for removal.
