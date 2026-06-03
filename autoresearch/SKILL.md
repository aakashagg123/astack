---
name: autoresearch
description: Autonomously optimize an existing Codex skill by benchmarking it across representative prompts, scoring outputs with binary evals, mutating the skill one change at a time, and keeping only improvements. Use when Codex needs to improve a skill, run repeated evals on a skill, benchmark prompt instructions, or produce a mutation log and dashboard for skill quality.
---

# Autoresearch

Use this skill to improve an existing skill through controlled experiments. Measure the current skill first, mutate one thing at a time, and keep only changes that improve results.

Treat this as an optimization loop, not a rewrite exercise. The goal is to produce a better skill plus a clear research record showing what was tried, what helped, and what still fails.

## Gather the required inputs

Before starting experiments, confirm these fields. Ask only for the missing ones.

- `target_skill`: exact path to the skill's `SKILL.md`
- `test_inputs`: 3-5 representative prompts or scenarios
- `evals`: 3-6 binary pass/fail checks
- `runs_per_experiment`: default `5`
- `run_interval`: default `2 minutes` for unattended cycles
- `budget_cap`: optional maximum experiment count

If the user does not provide an output filename for the improved variant, default to `<skill-name>-optimized.md`.

Use the user's eval criteria when they are usable. If they are vague, convert them into binary questions with help from [references/eval-guide.md](references/eval-guide.md).

## Read the target skill

Read the target skill before making changes.

1. Read the full `SKILL.md`.
2. Read any referenced files linked from that skill that are necessary to understand the workflow.
3. Summarize the skill's core job, expected outputs, reusable resources, and likely failure points.
4. Note anything that is already strong so you do not mutate it unnecessarily.

Do not start optimizing a skill you have not understood.

## Build the eval suite

Convert the success criteria into a stable pass/fail test set.

Use this format for each eval:

```text
EVAL [number]: [Short name]
Question: [Yes/no question]
Pass condition: [Specific condition that counts as yes]
Fail condition: [Specific condition that counts as no]
```

Rules:

- Use binary checks only.
- Make each check specific enough that two reviewers would likely agree.
- Avoid checks that can be gamed by superficial compliance.
- Keep the eval suite small enough to preserve signal. `3-6` evals is usually enough.
- Prefer a mix of structure, correctness, and user-value checks.

Calculate:

```text
max_score = number_of_evals * runs_per_experiment
```

If the evals are weak, fix the evals before mutating the skill.

## Create the working area

Never edit the original skill in place.

Create a working directory beside the target skill:

```text
autoresearch-[skill-name]/
```

Create these files:

```text
autoresearch-[skill-name]/
├── dashboard.html
├── results.json
├── results.tsv
├── changelog.md
├── SKILL.md.baseline
└── [optimized-filename].md
```

Required setup steps:

1. Copy the original `SKILL.md` to `SKILL.md.baseline`.
2. Copy the original `SKILL.md` to `[optimized-filename].md`. Mutate only this file.
3. Initialize `results.tsv` with a header row.
4. Initialize `results.json` so the dashboard can render immediately.
5. Generate a single-file `dashboard.html` that reads `results.json` and auto-refreshes.

If the environment supports opening local files and the user has not said otherwise, open `dashboard.html`. Otherwise report the path and continue.

## Establish the baseline

Run experiment `0` with the unmodified working copy.

1. Execute the target skill `runs_per_experiment` times across the fixed `test_inputs`.
2. Score every run against every eval.
3. Record the baseline in both `results.tsv` and `results.json`.
4. Write a short baseline note to `changelog.md`.

Use this `results.tsv` format:

```text
experiment	score	max_score	pass_rate	status	description
0	14	20	70.0%	baseline	original skill - no changes
```

If baseline performance is already above `90%`, ask the user whether to continue. The remaining work may have low return.

## Run the mutation loop

Once the baseline is established, optimize autonomously until a stop condition is hit.

For each experiment:

1. Analyze the failures.
2. Identify the most common failure pattern.
3. Form one hypothesis for one specific change.
4. Edit `[optimized-filename].md` with that one mutation.
5. Re-run the same experiment setup.
6. Score the results.
7. Keep or discard the change.
8. Update `results.tsv`, `results.json`, and `changelog.md`.

Use these keep/discard rules:

- `improved score`: keep the mutation and treat it as the new baseline
- `same score`: discard the mutation unless it meaningfully simplifies the skill
- `worse score`: discard the mutation

Good mutations:

- clarify an ambiguous instruction
- move a crucial instruction earlier in the skill
- add one concrete anti-pattern for a recurring failure
- replace vague wording with a precise requirement
- add one worked example if the skill lacks examples
- remove instructions that cause overfitting or conflicts

Bad mutations:

- rewrite the skill from scratch
- change many variables at once
- add long rules without a failure-driven reason
- optimize for a single eval while harming overall utility

If several mutations fail in a row, re-read the failing outputs before trying again.

## Prevent overfitting

Optimize for the real job, not only for the eval script.

- Keep the `test_inputs` varied.
- If practical, reserve one unseen prompt as a holdout check before declaring victory.
- Prefer simple instructions that generalize.
- Remove changes that only improve one narrow scenario.
- Treat unchanged output quality plus higher eval scores as a warning sign that the evals are weak.

If the skill starts parroting eval language instead of doing the task better, redesign the eval suite.

## Update the dashboard data

Keep `results.json` current after every experiment. Use this shape:

```json
{
  "skill_name": "skill-name",
  "status": "running",
  "current_experiment": 3,
  "baseline_score": 70.0,
  "best_score": 90.0,
  "experiments": [
    {
      "id": 0,
      "score": 14,
      "max_score": 20,
      "pass_rate": 70.0,
      "status": "baseline",
      "description": "original skill - no changes"
    }
  ],
  "eval_breakdown": [
    {
      "name": "Formatting correct",
      "pass_count": 8,
      "total": 10
    }
  ]
}
```

The dashboard should show:

- pass-rate trend across experiments
- keep vs discard status
- per-eval pass counts
- current run status
- a table of experiment summaries

When the optimization run ends, set `status` to `complete`.

## Stop conditions

Stop when one of these conditions is true:

- the user explicitly stops the run
- the budget cap is reached
- the skill reaches `95%+` pass rate for `3` consecutive experiments
- the eval suite is no longer trustworthy and must be rewritten

Do not stop between experiments just to ask for routine confirmation.

## Log every experiment

Append one entry per experiment to `changelog.md`:

```markdown
## Experiment [N] - [keep/discard]

**Score:** [X]/[max] ([percent]%)
**Change:** [One-sentence summary of the mutation]
**Reasoning:** [Why this mutation should help]
**Result:** [What improved or regressed]
**Remaining failures:** [What still fails, if anything]
```

Treat the changelog as a research artifact. Another agent should be able to continue from it without rereading the whole history of the conversation.

## Deliver the final result

When the run stops, report:

1. baseline score to final score
2. total experiment count
3. keep rate
4. top changes that helped most
5. remaining failure patterns
6. path to `[optimized-filename].md`
7. paths to `results.tsv`, `results.json`, `dashboard.html`, and `changelog.md`

Make it explicit that the original `SKILL.md` was not modified.

## Example mutation sequence

Use this pattern as a model:

1. Baseline the original skill.
2. Notice repeated formatting failures.
3. Add one explicit formatting rule.
4. Re-run and keep the rule only if the score improves.
5. Notice color or tone drift.
6. Replace vague style language with a precise example.
7. Re-run and keep only if the score improves.
8. Stop after the score plateaus and document the remaining edge cases.

The point is not to run many experiments. The point is to run disciplined experiments that produce a defensible improvement trail.
