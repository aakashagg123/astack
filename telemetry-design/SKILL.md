---
name: telemetry-design
description: >
  Structured logging, metrics, tracing, and alerting. Use this skill whenever the user asks to
  add logging, instrument a service, set up monitoring or alerts, or debug production behaviour
  that local reproduction can't reach. Also trigger proactively when shipping any new service,
  endpoint, job, or integration — instrumentation is designed with the feature, not bolted on
  after the first incident. The test: when this breaks at 3am, can the on-call person diagnose
  it from telemetry alone, without adding a print statement and redeploying?
---

# Telemetry Design

You are making a system diagnosable from the outside. The moment you need telemetry is the
moment you can no longer add it — production is on fire and redeploying with more logging is
the slowest possible debugger. Instrument *before* you need it.

**The gate: no new surface ships without its telemetry.** A service, endpoint, job, or
integration that can fail silently is not done — it is a future incident with no evidence
trail. Instrumentation is part of the change, not a follow-up ticket. The bar is the 3am
test: could on-call diagnose this failing from telemetry alone, without adding a log line and
redeploying? If no, it doesn't pass.

---

## The Three Signals

- **Logs** answer "what happened in this specific case" — discrete events with context.
- **Metrics** answer "how is the system behaving overall" — cheap aggregates over time
  (rates, latencies, saturation) that power dashboards and alerts.
- **Traces** answer "where did this request spend its time" — one request's path across
  functions and services.

Logs alone are not observability: you can't alert on grep, and you can't see a latency trend
in a stream of text.

---

## Logging Rules

1. **Structured, always.** Key-value/JSON events, not prose sentences. `"event":
   "payment_failed", "order_id": …, "reason": …` is queryable; "Payment failed for order!"
   is not.
2. **Correlated.** Every log line carries the request/job correlation ID so one request's
   story can be reassembled across services. Generate the ID at the edge; propagate it
   everywhere.
3. **Levelled honestly.** ERROR = something is wrong and needs action; WARN = surprising but
   handled; INFO = state changes worth an audit trail (job started, config loaded, order
   placed); DEBUG = development detail, off by default in production. An ERROR that nobody
   needs to act on trains people to ignore ERROR.
4. **Log the decision points.** Boundaries (requests in/out), state transitions, retries,
   fallbacks taken, and every caught exception — with enough context to act on. A caught-and-
   silently-swallowed exception is an unsolvable future incident.
5. **Never log secrets or PII.** No passwords, tokens, API keys, full card numbers, session
   cookies; redact or hash user identifiers per the project's privacy posture. Logs outlive
   databases and get copied everywhere — treat them as semi-public
   (pairs with `/secure-coding-practices`).

---

## Metrics Rules

- Instrument the **four golden signals** per service/endpoint: traffic (rate), errors,
  latency (as percentiles — p50/p95/p99, never averages, which hide the pain), and
  saturation (queue depth, pool usage, memory).
- Add domain metrics for what the feature is *for*: orders placed, messages delivered,
  jobs completed. A service can be technically green while the business metric flatlines —
  you want to see both.
- Keep label/tag cardinality bounded (no user IDs as metric labels).

---

## Alerting Rules

- Alert on **symptoms users feel** (error rate, p95 latency, backlog age), not on causes
  (CPU%) — causes go on dashboards for diagnosis.
- Every alert must be actionable: it names the threshold crossed and links to where to look.
  An alert with no action is noise, and noise gets muted — tune or delete flapping alerts
  the week they start flapping.
- Health/readiness endpoints for anything deployed; they gate rollouts in `/launch-gate`.

---

## Retrofitting an Opaque System

When debugging something with no telemetry: add the structured events you wished existed
(boundaries + decision points of the failing path first), keep them after the incident —
the next incident will be in the neighbourhood — and only then dive into `/root-cause`.

---

## Anti-Patterns to Refuse

- **The silent catch.** `catch (e) {}` — swallowing an exception with no log is deleting the
  one clue the future incident will need.
- **Prose logs.** "Payment failed for order!" — ungreppable, un-alertable. Structured
  key-value events only.
- **Averages instead of percentiles.** A mean latency of 200ms hides the p99 of 8s that users
  actually feel. Percentiles or it didn't happen.
- **Alerting on causes.** Paging on CPU% instead of the error rate or latency users
  experience — causes belong on dashboards, symptoms on pagers.
- **Secrets in logs.** Tokens, passwords, full PANs, session cookies. Logs outlive databases
  and leak everywhere; this is a security incident waiting in plaintext.
- **"We'll add logging when we need it."** By then production is down and you're redeploying
  blind. That is exactly the moment this skill exists to prevent.

---

## Exit Criteria

1. The 3am test passes: failure modes of this change are diagnosable from logs/metrics/traces
   without a redeploy.
2. Events structured and correlation-ID'd; levels honest; zero secrets/PII in any log path.
3. Golden-signal metrics exist for new surfaces; alerts (if any) are symptom-based and
   actionable.
4. Instrumentation shipped in the same change as the feature — not filed as "later".
