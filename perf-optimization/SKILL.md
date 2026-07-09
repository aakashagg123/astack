---
name: perf-optimization
description: >
  Measurement-first performance work. Use this skill whenever the user says something is slow,
  asks to optimise, mentions Core Web Vitals, bundle size, memory, latency, or throughput —
  or when a performance budget or SLO is being missed. Also trigger proactively before agreeing
  to any "quick perf tweak": the skill's first job is to demand a measurement, because most
  perceived bottlenecks are not where intuition says they are. Never optimise on vibes; never
  claim a speed-up without a before/after number.
---

# Performance Optimization

You are doing evidence-driven performance work. The iron rule: **no measurement, no
optimization**. An optimization without a before/after number is a style change with risk
attached.

---

## Step 1 — Define the Target

Before profiling, pin down:
- **The metric** — what exactly is slow? (p95 latency, LCP, INP, CLS, bundle bytes, memory,
  cold start, query time). "Feels slow" must be converted into one named metric.
- **The budget** — what number counts as fixed? Get it from the user or propose one
  (e.g. web defaults: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1; API p95 within SLO).
- **The conditions** — device class, network, data volume, cache state. A fix measured on a
  warm dev machine with 10 rows proves nothing about production with 10 million.

---

## Step 2 — Measure and Locate

- Profile with the right instrument: browser performance panel / Lighthouse for web vitals,
  a profiler or flame graph for CPU, `EXPLAIN` for queries, bundle analyzer for payload,
  heap snapshots for memory.
- Record the **baseline** — exact numbers, exact conditions — before changing anything.
- Find the dominant cost. Optimizing anything below ~20% of total cost caps your best-case
  win below 20%; go after the biggest bar in the flame graph first.

---

## Step 3 — Fix in Order of Leverage

Work down this ladder; each rung beats the ones below it:

1. **Do less work** — eliminate the call, cache the result, dedupe repeated computation,
   paginate instead of fetching everything.
2. **Do work later or elsewhere** — lazy-load, defer below-the-fold, move off the critical
   path, background the batch job.
3. **Do work faster** — better algorithm or data structure, index the query, batch the N+1.
4. **Micro-optimize** — only after 1–3 are exhausted, and only with a benchmark proving it
   matters.

Common web-specific wins, in typical payoff order: eliminate render-blocking resources,
compress/resize images and set dimensions, code-split and tree-shake the bundle, cache
aggressively with correct invalidation, reduce main-thread JS during interaction.

---

## Step 4 — Verify and Guard

- Re-measure under the **same conditions** as the baseline. Report both numbers.
- Confirm correctness didn't regress: full test suite green (a fast wrong answer is worse
  than a slow right one).
- Add a regression guard where practical: performance budget in CI, Lighthouse assertion,
  benchmark test, query-time alert.
- If the change added complexity, record why in a comment tied to the number
  ("batched: 40→1 queries, p95 900ms→80ms") so a future `/code-simplify` pass doesn't undo it.

---

## Rationalizations to Refuse

- "This is obviously faster" — measure it. Obvious optimizations lose to compilers, caches,
  and JIT behaviour constantly.
- "While I'm here, I'll optimize this too" — unmeasured scope creep. One target per pass.
- "It's faster on my machine" — that's one condition, and the least representative one.
- "We'll need this at scale" — speculative optimization is complexity paid today for a load
  profile that may never arrive. Note it, don't build it.

---

## Exit Criteria

1. Baseline and post-change measurements reported side by side, same conditions.
2. Target metric within budget, or an explicit statement of the remaining gap and next lever.
3. Test suite green; behaviour unchanged.
4. Regression guard in place, or an explicit note on why one isn't practical.
