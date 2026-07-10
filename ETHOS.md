# ETHOS

## What astack optimises for

**Discovery before code.** The most expensive engineering mistake is building the right solution to the wrong problem. `/problem-discovery` is the mandatory first step for any work where the solution space is open. Six phases of structured questioning, codebase scanning, and alternative generation — before a single line of spec is written.

**Spec before sprint.** A spec that hasn't been pressure-tested isn't a spec — it's a wish. `/spec-creator` produces a complete document with acceptance criteria, a decisions register, an assumption register, and test cases that map 1:1 to scenarios. The spec is the contract. The implementation follows it, not the other way around.

**TDD as default, not exception.** `/red-green-tdd` enforces the discipline strictly: RED (write a failing test), GREEN (make it pass with minimal code), REFACTOR (clean without breaking). No shortcuts. The test file is written before the implementation file, every time.

**Security before code ships.** `/secure-coding-practices` runs before any auth, API key, or access-control change. Not as a post-hoc audit — as a pre-implementation posture check. Security problems found in review cost 10× what they cost to prevent.

**Review against requirements, not just style.** `/pr-review` checks code against the original spec and acceptance criteria, not just linting and naming conventions. A PR that passes ESLint but ships the wrong behaviour is still a failed PR.

**Cycle closure is not optional.** `/context-sync` closes every dev cycle by updating the project's working memory with what was learned. Patterns that worked, gotchas to avoid, decisions made. The next session starts from a richer context, not from scratch.

**Root cause before patch.** When something breaks, `/root-cause` treats it as a case to solve — evidence, diagnosis, cure, immunity — and halts new work until the diagnosis holds. Making the error disappear is not the goal; understanding why it existed is. A fix you can't explain is a coincidence.

**Evidence over recall, runtime over reading.** `/docs-grounding` verifies framework and library usage against official docs for the version actually installed — memory of an API is a snapshot that started aging the day it was taken. `/browser-verify` holds the same line for UI: a change verified only by reading the diff is unverified. And `/perf-tuning` refuses any speed claim without a before/after number.

**Shipping should be boring.** `/launch-gate` gates every release on three properties: verified before exposure, observable during exposure, reversible at every stage. `/telemetry-design` makes systems diagnosable before the incident, not after. `/git-hygiene` keeps the history a readable record of why. Excitement at deploy time is a process failure.

## Growth is a build cycle too

The same discipline that prevents building the wrong feature prevents shipping the wrong campaign. astack's growth pipeline applies the build ethos to the product-to-market arc — for the product managers and product marketers who own what happens *after* the feature ships.

**Discovery before campaign.** `/growth-discovery` maps the lay of the land — martech stack, budget, personas, user scale, channel mix — before any brief is written. A campaign without a named growth motion is a tactic looking for a strategy. This is the marketing twin of `/problem-discovery`.

**The brief is the contract.** `/marketing-brief` turns the strategy into a brief grounded in product analytics and the features that have *actually shipped* — never the roadmap. Every message maps to a live truth; every objective maps to a measurable KPI with a baseline and a guardrail. Marketing ahead of the product burns trust. The brief is the marketing twin of the spec.

**Data closes the loop.** `/campaign-insights` ingests past campaign, messaging, and web-analytics data and returns a Brief Delta — prioritised, evidence-backed changes, not a dashboard of vanity charts. An insight that doesn't change the brief is trivia. This is the marketing twin of cycle closure.

**The journey is the artifact.** `/marketing-automation-builder` turns an approved brief into an executable customer journey, rendered as a beautified, standalone HTML workflow with KPIs wired in. Every node traces back to a brief objective. The artifact is portable and stack-neutral — it uses a clean, generic design system, never a client's.

## What astack is not

**Not a code executor.** astack skills don't write code for you on their own — they structure the problem, generate the plan, and coordinate workers. The goal is better output through better process, not automation for its own sake.

**Not opinionated about your stack.** The skills work with whatever language, framework, or architecture you're using. There are no React-specific or Python-specific assumptions baked in. Adapt the patterns to your project.

**Not a replacement for judgment.** The skills surface structure, surface alternatives, surface tradeoffs. The decision is always yours. `/evaluate-approach` gives you three options with scored tradeoffs — it doesn't pick for you.

**Not your martech platform.** The growth pipeline structures the thinking that feeds your automation tool — it doesn't replace your CDP, ESP, or journey engine. It produces the strategy, the brief, the insights, and the workflow design. Execution still happens in your stack.

## Where astack fits

astack now spans two full cycles. The **build cycle** runs end to end: problem discovery, spec writing, story decomposition, architecture planning on the front half — then sliced implementation, root-cause debugging, runtime verification, review, and staged shipping on the back half. The **growth cycle** covers strategy mapping, brief writing, data-led refinement, and journey design — what gets shipped to market before any send goes out.

Pair it with whatever execution layer you already use — code agents and your own harness on the build side, your martech and automation platforms on the growth side. astack is designed to hand off cleanly into any of them.
