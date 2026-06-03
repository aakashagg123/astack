---
name: preflight
description: "Validates environment before executing any task. Checks CLI tools (node, npm, python, aws, gh, etc.), API auth tokens, disk space, network connectivity, and env config — then writes preflight-report.md. Blocks execution if any check fails; proposes fixes and waits for approval. Trigger on: \"/preflight\", \"run preflight\", \"check my environment\", \"preflight check\", \"validate setup\", \"are we ready to\", \"check dependencies\". Also trigger proactively before deployments, migrations, pipelines, build scripts, or any task involving external APIs, S3, databases, or CLI tools — especially before /red-green-tdd, /secure-coding-practice, or deployment scripts. Trigger on execution-intent phrases: \"deploy to prod\", \"run the migration\", \"kick off the build\", \"run the script\". Never proceed optimistically past a failed check."
---

# Preflight — Skill Guide

Catches environment failures before they waste execution time. One failed API key or missing
CLI tool mid-task costs 10x more than a 60-second preflight check at the start. This skill
makes environment validation a first-class step, not an afterthought.

---

## Behaviour rules

**Rule 1 — Always run before proceeding.**
Preflight is not optional. If triggered, run all applicable checks before any task work
begins. Do not partially run checks and optimistically proceed.

**Rule 2 — Binary results per check.**
Every check is PASS ✅, FAIL ❌, or SKIP ⭐ (with reason). No "probably fine" verdicts.

**Rule 3 — Blockers stop execution.**
If any check returns FAIL, do not proceed with the task. Propose fixes, list them clearly,
and wait for explicit user confirmation before retrying or continuing.

**Rule 4 — Write the report, always.**
Output is always written to `preflight-report.md` in the working directory, regardless
of pass/fail outcome. The report is the audit trail.

**Rule 5 — Scope checks to the task.**
Run only checks relevant to the stated task. A Python data script doesn't need `gh` or
`aws` checks. A deployment pipeline doesn't need disk space checked on the wrong volume.
Infer scope from the task description; ask if ambiguous.

**Rule 6 — Propose fixes, don't auto-fix.**
If a check fails, propose the exact fix command or action. Do not execute fixes without
user approval. The user may have environment-specific reasons for a configuration.

**Rule 7 — Re-run on request.**
After user applies fixes, re-run only the failed checks (not the full suite) and update
the report. Mark re-checked items clearly.

---

## Intake

Before running checks, confirm from context or ask:

1. What is the task to be run? (describe in one sentence)
2. Which services/APIs does it touch? (S3, RDS, Supabase, OpenAI, credit bureau API, telephony API, etc.)
3. What is the working directory / repo?
4. Are there environment-specific config files (.env, config.yaml) to validate?
5. Are there known constraints on disk space or file size for this task?

If the task is clear from context, skip asking and infer — state your assumptions in
the report header.

---

## Check catalogue

Run applicable checks from these categories. Mark non-applicable checks as SKIP with reason.

### Category 1 — CLI tools

For each tool required by the task, run: `<tool> --version` or equivalent.

| Tool | Check command | Required for |
|---|---|---|
| node | `node --version` | JS/TS builds, npm scripts |
| npm | `npm --version` | Package management |
| npx | `npx --version` | Script runners |
| python | `python --version` or `python3 --version` | Python scripts, ML |
| pip | `pip --version` | Python packages |
| gh | `gh auth status` | GitHub CLI operations |
| git | `git --version` | Repo operations |
| aws | `aws --version` + `aws sts get-caller-identity` | AWS operations |
| docker | `docker --version` + `docker info` | Container builds |
| kubectl | `kubectl version --client` | K8s deployments |
| terraform | `terraform --version` | Infra provisioning |
| curl | `curl --version` | HTTP checks |
| jq | `jq --version` | JSON processing |

Add task-specific tools not listed here.

### Category 2 — Auth and API keys

For each external service in scope:

- **Existence check**: Is the env var / secret present? (`echo $VAR_NAME` — show masked)
- **Validity check**: Make a minimal test call (list, ping, whoami — not a mutating call)
- **Permissions check**: Does the token have the required scope/role?

Common checks:

| Service | Validity test |
|---|---|
| AWS | `aws sts get-caller-identity` |
| GitHub | `gh auth status` |
| OpenAI / Anthropic | `curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $KEY" <ping-endpoint>` |
| Supabase | Test select on a known table |
| telephony API | Health endpoint ping |
| credit bureau API | Connectivity to API gateway (no live data call) |
| Custom APIs | GET /health or /ping on base URL |

Never log or display raw API keys. Mask all but last 4 characters.

### Category 3 — Disk space and file constraints

- Available disk space on working volume: `df -h .`
- Compare against estimated task output size (state estimate explicitly)
- Check if input files exist and are within expected size bounds
- Check for write permissions on output directory: `touch <dir>/.preflight_write_test && rm <dir>/.preflight_write_test`

Thresholds:
- WARN if available < 2x estimated task output
- FAIL if available < 1.1x estimated task output

### Category 4 — Network connectivity

For each external endpoint the task requires:

```bash
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 <endpoint>
```

- HTTP 200/2xx = PASS
- Timeout or non-2xx = FAIL (note: auth errors are auth failures, not network failures)
- Check DNS resolution separately if base connectivity is suspect: `nslookup <domain>`

### Category 5 — Environment config

- Does `.env` / config file exist at expected path?
- Are all required keys present (check key names, not values)?
- Are there conflicting environment variables (e.g., two AWS profiles active)?
- Is the correct git branch checked out (if task is branch-specific)?
- Is the correct Python venv / node_modules present?

---

## Report format

Write to `preflight-report.md`:

```markdown
# Preflight Report

**Task:** <one-line task description>
**Timestamp:** <ISO datetime>
**Working directory:** <path>
**Assumptions:** <any inferred from context>

---

## Summary

| Category | Status | Checks Passed | Blockers |
|---|---|---|---|
| CLI Tools | ✅ PASS / ❌ FAIL | N/N | N |
| Auth & API Keys | ✅ PASS / ❌ FAIL | N/N | N |
| Disk & Files | ✅ PASS / ❌ FAIL | N/N | N |
| Network | ✅ PASS / ❌ FAIL | N/N | N |
| Environment Config | ✅ PASS / ❌ FAIL | N/N | N |

**Overall: ✅ CLEAR TO PROCEED / ❌ BLOCKED**

---

## Detail

### CLI Tools
| Tool | Version | Status | Notes |
|---|---|---|---|
| node | v20.11.0 | ✅ | |
| aws | 2.15.0 | ❌ | Not installed |
| gh | ⭐ SKIP | Not required for this task | |

### Auth & API Keys
| Service | Key Present | Valid | Permissions | Status |
|---|---|---|---|---|
| AWS | ✅ | ✅ | S3:ReadWrite | ✅ |
| OpenAI | ✅ | ❌ | ❌ | ❌ Key rejected (401) |

### Disk & File Constraints
- Available: Xgb on <volume>
- Estimated task output: ~Xmb
- Input files: <list with sizes>
- Write test: ✅ / ❌
- **Status: ✅ / ❌**

### Network Connectivity
| Endpoint | HTTP Status | Latency | Status |
|---|---|---|---|
| api.openai.com | 200 | 142ms | ✅ |
| s3.amazonaws.com | ❌ | timeout | ❌ |

### Environment Config
| Check | Status | Notes |
|---|---|---|
| .env file present | ✅ | |
| All required keys | ❌ | Missing: DATABASE_URL |
| Git branch | ✅ | main |

---

## Blockers

> Found N blocker(s). Task is HELD pending resolution.

1. **[BLOCKER]** `aws` CLI not installed.
   Fix: `brew install awscli` or `pip install awscli`

2. **[BLOCKER]** `OPENAI_API_KEY` returns 401.
   Fix: Regenerate key at platform.openai.com → Settings → API Keys. Update `.env`.

3. **[BLOCKER]** `DATABASE_URL` missing from `.env`.
   Fix: Add `DATABASE_URL=<connection-string>` to `.env`.

---

## Proceed?

All blockers must be resolved before task execution begins.
Confirm fixes applied — preflight will re-run failed checks only.
```

---

## Re-run protocol

After user confirms fixes:

1. Re-run only FAIL checks (not the full suite)
2. Update `preflight-report.md` in place — mark re-checked items with `[re-checked ✅]`
3. If all previously-failed checks now pass — declare CLEAR TO PROCEED
4. If any still fail — repeat blocker list, wait again

---

## Confidence score

After writing the report, state in chat:
`Preflight confidence: X% — [N] checks run, [N] passed, [N] failed, [N] skipped.`

Below 80%: explain which check category had limited visibility and why.