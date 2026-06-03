# ETHOS

## What astack optimises for

**Discovery before code.** The most expensive engineering mistake is building the right solution to the wrong problem. `/problem-discovery` is the mandatory first step for any work where the solution space is open. Six phases of structured questioning, codebase scanning, and alternative generation — before a single line of spec is written.

**Spec before sprint.** A spec that hasn't been pressure-tested isn't a spec — it's a wish. `/spec-creator` produces a complete document with acceptance criteria, a decisions register, an assumption register, and test cases that map 1:1 to scenarios. The spec is the contract. The implementation follows it, not the other way around.

**TDD as default, not exception.** `/red-green-tdd` enforces the discipline strictly: RED (write a failing test), GREEN (make it pass with minimal code), REFACTOR (clean without breaking). No shortcuts. The test file is written before the implementation file, every time.

**Security before code ships.** `/secure-coding-practices` runs before any auth, API key, or access-control change. Not as a post-hoc audit — as a pre-implementation posture check. Security problems found in review cost 10× what they cost to prevent.

**Review against requirements, not just style.** `/pr-review` checks code against the original spec and acceptance criteria, not just linting and naming conventions. A PR that passes ESLint but ships the wrong behaviour is still a failed PR.

**Cycle closure is not optional.** `/context-sync` closes every dev cycle by updating the project's working memory with what was learned. Patterns that worked, gotchas to avoid, decisions made. The next session starts from a richer context, not from scratch.

## What astack is not

**Not a code executor.** astack skills don't write code for you on their own — they structure the problem, generate the plan, and coordinate workers. The goal is better output through better process, not automation for its own sake.

**Not opinionated about your stack.** The skills work with whatever language, framework, or architecture you're using. There are no React-specific or Python-specific assumptions baked in. Adapt the patterns to your project.

**Not a replacement for judgment.** The skills surface structure, surface alternatives, surface tradeoffs. The decision is always yours. `/evaluate-approach` gives you three options with scored tradeoffs — it doesn't pick for you.

## Where astack fits in a build workflow

astack covers the front half of the build cycle: problem discovery, spec writing, story decomposition, architecture planning. The work that determines *what* gets built before any implementation begins.

Pair it with whatever execution layer you already use — code agents, manual implementation, or your own harness. astack is designed to hand off cleanly into any of them.
