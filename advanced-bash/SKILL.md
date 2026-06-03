---
name: advanced-bash
description: >
  Enhanced shell command execution for developers. Invoke this skill whenever
  you need to: run complex multi-step shell pipelines, execute build/test/lint
  commands with proper error handling, manage background processes and ports,
  chain dependent commands safely, handle environment variables and secrets,
  run database migrations or CLI tools, orchestrate Docker/container operations,
  manage git operations beyond simple status/log, parse and transform command
  output, or when a shell task requires more than a simple one-liner.
  Trigger phrases: "run this pipeline", "build and test", "chain these commands",
  "manage this process", "set up the environment", "run the migration",
  "docker compose", "kill the process on port", "parse this output".
  Always prefer this skill over raw Bash for multi-step or error-prone shell tasks.
---

# Advanced Bash

The core principle: **The Bash tool is the execution engine — but most file
reading, searching, and editing has a better dedicated tool.** Use Bash for
running commands, pipelines, builds, tests, process management, and anything
that produces side effects. Never use it as a substitute for Read, Edit, Write,
Grep, or Glob.

This skill is the third in a trio:
- **advanced-grep** — content search (`rg`, `jq`, symbol correlation)
- **advanced-glob** — file finding (`fd`, name patterns, recency)
- **advanced-bash** — shell execution (this skill)

---

## Tool Selection Matrix

| Task | Tool | NOT this |
|------|------|----------|
| Run build / test / lint | **Bash** | — |
| Chain dependent commands | **Bash** with `&&` | — |
| Kill a process / manage ports | **Bash** (`lsof`, `kill`) | — |
| Start a background server | **Bash** with `run_in_background` | — |
| Check env vars / validate config | **Bash** (`printenv`) | — |
| Git commit, rebase, cherry-pick | **Bash** (`git`) | — |
| Docker compose, exec, logs | **Bash** (`docker`) | — |
| Parse JSON output | **Bash** (`jq`) | — |
| Run a database migration | **Bash** (CLI tool) | — |
| Read a file's contents | **Read** tool | Bash `cat`/`head`/`tail` |
| Search file contents by pattern | **Grep** tool | Bash `grep`/`rg` |
| Find files by name/glob | **Glob** tool | Bash `find`/`ls` |
| Edit a file (replace text) | **Edit** tool | Bash `sed`/`awk` |
| Write a new file | **Write** tool | Bash `echo >` |
| Stream background output | **Monitor** tool | Bash `sleep` loops |

**Decision rule:** If the command produces a side effect (runs a process,
changes system state, installs something, triggers a build), use Bash. If
the command only reads or transforms file content, use the dedicated tool.

---

## Playbook 1 — Multi-Step Command Chaining

### Operators and when to use them

```bash
# && — run B only if A succeeds (most common, safest default)
npm install && npm run build && npm run test

# ; — run B regardless of whether A succeeds (use for cleanup)
npm run test; echo "Tests finished (pass or fail)"

# || — run B only if A fails (error recovery / fallback)
npm run build || echo "BUILD FAILED" && exit 1

# Subshell — isolate side effects (cd, env changes)
(cd /tmp && git clone repo.git && cd repo && npm install)
# cwd is unchanged after the subshell exits
```

### Error propagation patterns

```bash
# Pattern 1: Fail fast — stop at first error
set -e && npm install && npm run build && npm run test

# Pattern 2: Capture exit code for conditional logic
npm run build; BUILD_EXIT=$?; if [ $BUILD_EXIT -ne 0 ]; then echo "Build failed with code $BUILD_EXIT"; fi

# Pattern 3: Multiple independent checks, report all failures
FAILURES=0
npm run lint || FAILURES=$((FAILURES+1))
npm run typecheck || FAILURES=$((FAILURES+1))
npm run test || FAILURES=$((FAILURES+1))
echo "Total failures: $FAILURES"
```

### When to use `set -e`

Use `set -e` when you want the entire pipeline to abort on first failure:
```bash
set -e && mkdir -p dist && npm run build && cp -r dist/* /deploy/
```

Do NOT use `set -e` when you need to inspect exit codes or continue after
partial failures — it turns every non-zero exit into an abort.

### Subshell isolation

The Bash tool resets working directory between calls. Within a single call,
use subshells to avoid cd side effects:

```bash
# BAD — cd affects subsequent commands in the same call
cd /tmp && do_thing && cd /other && do_other_thing

# GOOD — isolated
(cd /tmp && do_thing) && (cd /other && do_other_thing)
```

### Piping vs chaining

```bash
# Pipe — stdout of A becomes stdin of B (data flow)
npm run test 2>&1 | tail -20

# Chain — sequential execution (control flow)
npm run test && npm run build
```

When piping, the exit code is from the LAST command in the pipe by default.
Use `set -o pipefail` if you need the pipe to fail when ANY command fails:
```bash
set -o pipefail && npm run test 2>&1 | tee test-output.log
```

---

## Playbook 2 — Build / Test / Lint Pipelines

### Node.js (npm / pnpm / yarn)

```bash
# Full CI pipeline
npm ci && npm run lint && npm run typecheck && npm run test && npm run build

# Run a single test file
npm run test -- src/lib/myModule.test.ts

# Run tests matching a pattern
npm run test -- --grep "should handle edge case"

# Watch mode (use run_in_background)
# Set run_in_background: true for this
npm run test:watch

# Check what would be published
npm pack --dry-run

# Audit dependencies
npm audit --production
```

### Python (pip / pytest / uv)

```bash
# Virtual env setup
python3 -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]"

# Run tests with verbose output
python3 -m pytest -v tests/

# Run a specific test
python3 -m pytest tests/test_module.py::test_specific_function -v

# Coverage
python3 -m pytest --cov=src --cov-report=term-missing tests/

# Type checking
python3 -m mypy src/ --ignore-missing-imports

# uv (fast alternative)
uv pip install -e ".[dev]" && uv run pytest
```

### Rust (cargo)

```bash
# Build + test + clippy
cargo build && cargo test && cargo clippy -- -D warnings

# Run a specific test
cargo test test_name -- --nocapture

# Release build
cargo build --release
```

### Capturing and interpreting exit codes

```bash
# Run tests, capture result, report
npm run test 2>&1; TEST_EXIT=$?
if [ $TEST_EXIT -eq 0 ]; then
  echo "ALL TESTS PASSED"
else
  echo "TESTS FAILED (exit code: $TEST_EXIT)"
fi
```

### Parallel test runs

```bash
# Node — most test runners support parallel by default
npm run test -- --reporter=verbose

# Python — pytest-xdist for parallel
python3 -m pytest -n auto tests/

# Run two independent checks in parallel within one Bash call
npm run lint & npm run typecheck & wait
```

**Better pattern for parallelism:** Make two separate Bash tool calls in the
same message — the tool infrastructure runs them in parallel automatically.
This is cleaner than backgrounding within a single call.

---

## Playbook 3 — Process and Port Management

### Finding what's running on a port

```bash
# macOS — find process on port 8080
lsof -i :8080

# Linux — find process on port 8080
ss -tlnp | grep 8080
# or
lsof -i :8080

# Cross-platform — just the PID
lsof -ti :8080
```

### Killing a process on a port

```bash
# Kill whatever is on port 8080
lsof -ti :8080 | xargs kill -9

# Safer — check first, then kill
PID=$(lsof -ti :8080)
if [ -n "$PID" ]; then
  echo "Killing PID $PID on port 8080"
  kill -9 $PID
else
  echo "Port 8080 is free"
fi
```

### Checking if a server is ready

```bash
# Wait for a server to become available (max 30 seconds)
for i in $(seq 1 30); do
  curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/health | grep -q "200" && echo "Server ready" && break
  sleep 1
done
```

**But prefer this pattern instead:** Start the server with `run_in_background: true`,
then use a separate Bash call with `timeout: 30000` to check readiness:

```bash
# Call 1: start server (run_in_background: true)
npm run dev

# Call 2: verify it's up (separate Bash call, timeout: 30000)
curl -sf http://localhost:8080/health
```

### Using `run_in_background`

Use `run_in_background: true` for:
- Dev servers (`npm run dev`, `uvicorn`, `python -m http.server`)
- Watch-mode test runners (`npm run test:watch`)
- Long-running builds you don't need to block on
- Any process that should keep running after the command returns

Do NOT use `run_in_background` for:
- Commands whose output you need immediately
- Commands where exit code matters for the next step
- Quick commands (< 5 seconds)

```bash
# Start dev server in background
# Set run_in_background: true, timeout: 120000
npm run dev

# Later — check if it's running
lsof -i :8080
```

### Process listing and management

```bash
# Find all node processes
ps aux | grep '[n]ode'

# Find process by name (bracket trick avoids matching grep itself)
ps aux | grep '[u]vicorn'

# Kill all node processes (careful!)
pkill -f node

# Kill a specific process tree
kill -TERM $PID
```

---

## Playbook 4 — Environment and Secrets

### Checking environment variables

```bash
# Check if a specific var is set and non-empty
printenv VITE_SUPABASE_URL || echo "VITE_SUPABASE_URL is not set"

# Check multiple vars at once
for VAR in VITE_SUPABASE_URL VITE_SUPABASE_PUBLISHABLE_KEY VITE_AUTH_PROVIDER; do
  if [ -z "$(printenv $VAR)" ]; then echo "MISSING: $VAR"; else echo "OK: $VAR"; fi
done

# List all env vars with a prefix (safe — shows names only, not values)
printenv | grep "^VITE_" | cut -d= -f1

# Show value (only for non-secret vars)
printenv VITE_AUTH_PROVIDER
```

### Secret safety rules

**NEVER do these:**
```bash
# NEVER echo a secret to stdout
echo $API_KEY                    # exposed in tool output
printenv SECRET_TOKEN            # exposed in tool output
cat .env                         # exposes all secrets

# NEVER pass secrets as command arguments (visible in ps)
curl -H "Authorization: Bearer $SECRET" https://api.example.com
```

**DO this instead:**
```bash
# Check existence without revealing value
[ -n "$API_KEY" ] && echo "API_KEY is set (length: ${#API_KEY})" || echo "API_KEY is NOT set"

# Validate format without revealing content
echo "$API_KEY" | grep -qE '^sk-[a-zA-Z0-9]{20,}$' && echo "API_KEY format looks valid" || echo "API_KEY format unexpected"

# Use env vars in config files, not CLI args
# Write to a temp config instead of passing on command line
```

### `.env` file validation

```bash
# Check that .env exists and has expected keys (without revealing values)
if [ -f .env ]; then
  echo ".env exists"
  for KEY in VITE_SUPABASE_URL VITE_SUPABASE_PUBLISHABLE_KEY; do
    grep -q "^${KEY}=" .env && echo "  $KEY: present" || echo "  $KEY: MISSING"
  done
else
  echo ".env does NOT exist"
fi

# Check for common .env problems
grep -n "^[A-Z].*= " .env     # space after = (usually wrong)
grep -n "^#" .env | wc -l     # count comment lines
grep -c "^[A-Z]" .env         # count actual var definitions
```

### Loading env for a command

```bash
# Source .env for the current command (simple vars only)
export $(grep -v '^#' .env | xargs) && npm run dev

# Safer — use env-cmd or dotenv-cli if available
npx env-cmd npm run dev
```

---

## Playbook 5 — Git Operations

### Fundamental constraints

- **NEVER use `-i` flag** (interactive rebase, interactive add — not supported)
- **NEVER use `--no-verify`** unless the user explicitly requests it
- **NEVER force-push to main/master** — warn the user if they request it
- **Always create NEW commits** rather than amending (unless user says amend)
- **Always use absolute paths** in git commands

### Safe commit pattern

```bash
# Stage specific files (never git add -A blindly)
git add src/components/MyComponent.tsx src/lib/myUtil.ts

# Commit with heredoc for proper formatting
git commit -m "$(cat <<'EOF'
feat: add validation to loan stage 3

Adds cross-field validation rules for financial data fields.
Prevents submission when DSCR is below threshold.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```

### Branch operations

```bash
# Create and switch to a new branch
git checkout -b feature/my-feature

# Push with upstream tracking
git push -u origin feature/my-feature

# Check if branch is up to date with remote
git status -sb

# List branches sorted by last commit
git branch --sort=-committerdate | head -10

# Delete a merged branch (local)
git branch -d feature/old-branch

# Fetch and prune stale remote refs
git fetch --prune
```

### Stash management

```bash
# Stash with a descriptive name
git stash push -m "WIP: stage 3 validation before switching to hotfix"

# List stashes
git stash list

# Apply most recent stash (keep it in stash list)
git stash apply

# Apply and drop
git stash pop

# Apply a specific stash
git stash apply stash@{2}

# Show what's in a stash
git stash show -p stash@{0}
```

### Cherry-pick patterns

```bash
# Cherry-pick a single commit
git cherry-pick abc123

# Cherry-pick without auto-committing (inspect first)
git cherry-pick --no-commit abc123
git diff --cached    # review what was picked
git commit -m "cherry-pick: description"

# Cherry-pick a range (oldest..newest, oldest is EXCLUDED)
git cherry-pick abc123^..def456

# Abort a failed cherry-pick
git cherry-pick --abort
```

### Diff and log for context

```bash
# Diff between branches (what will the PR contain)
git diff main...HEAD

# Log with file list
git log --oneline --name-only -10

# Log for a specific file
git log --oneline -20 -- src/store/loanStore.tsx

# Find the commit that introduced a string
git log -S "functionName" --oneline

# Show what changed in a specific commit
git show abc123 --stat
git show abc123 -- src/specific/file.ts
```

### Merge conflict resolution

```bash
# After a failed merge, see what's conflicted
git status --short | grep "^UU"

# After manually fixing conflicts (via Edit tool), mark resolved
git add src/resolved-file.ts
git commit -m "merge: resolve conflicts in resolved-file.ts"

# Abort a merge
git merge --abort

# Abort a rebase
git rebase --abort
```

### Worktree operations

```bash
# Create a worktree for parallel work
git worktree add ../my-worktree feature-branch

# List worktrees
git worktree list

# Remove a worktree
git worktree remove ../my-worktree
```

---

## Playbook 6 — Docker and Containers

### Compose operations

```bash
# Start services (detached)
docker compose up -d

# Start specific services only
docker compose up -d postgres redis

# Rebuild and restart
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes (destructive — confirm first)
docker compose down -v

# View logs
docker compose logs -f --tail=50

# View logs for one service
docker compose logs -f --tail=100 api
```

### Container interaction

```bash
# Exec into a running container
docker compose exec api bash
docker compose exec api sh   # if bash isn't available

# Run a one-off command in a service
docker compose exec db psql -U postgres -d mydb -c "SELECT count(*) FROM users;"

# Run a migration inside the container
docker compose exec api alembic upgrade head
```

### Image management

```bash
# Build an image
docker build -t myapp:latest .

# List images
docker images | grep myapp

# Remove dangling images (safe cleanup)
docker image prune -f

# Remove ALL unused images (aggressive — confirm first)
docker image prune -a -f
```

### Health checks and debugging

```bash
# Check container status
docker compose ps

# Check container resource usage
docker stats --no-stream

# Inspect a container
docker inspect $(docker compose ps -q api) | jq '.[0].State'

# Check if a containerized service is healthy
docker compose exec api curl -sf http://localhost:8000/health || echo "UNHEALTHY"

# View container logs since a timestamp
docker compose logs --since="5m" api
```

### Network debugging

```bash
# List networks
docker network ls

# Inspect a network (see connected containers)
docker network inspect $(docker compose ps -q api | head -1) 2>/dev/null || docker network inspect bridge

# Test connectivity between containers
docker compose exec api ping -c 3 db
```

---

## Playbook 7 — Output Parsing and Transformation

### JSON with `jq`

```bash
# Pretty-print
echo '{"a":1,"b":2}' | jq .

# Extract a field
echo '{"name":"loan","status":"active"}' | jq -r '.status'

# Filter array elements
echo '[{"id":1,"ok":true},{"id":2,"ok":false}]' | jq '.[] | select(.ok == true)'

# Map array to specific fields
echo '[{"id":1,"name":"a"},{"id":2,"name":"b"}]' | jq '.[].name'

# Count array elements
echo '[1,2,3,4,5]' | jq 'length'

# Nested access
echo '{"data":{"items":[{"v":1},{"v":2}]}}' | jq '.data.items[].v'

# Parse npm/package.json
jq '.dependencies | keys' package.json

# Parse tsconfig
jq '.compilerOptions.paths' tsconfig.json
```

### Columnar data with `awk` and `cut`

```bash
# Extract specific columns from space-separated output
ps aux | awk '{print $1, $2, $11}'    # user, pid, command

# Sum a numeric column
awk '{sum += $5} END {print sum}' data.txt

# Filter rows by column value
awk '$3 > 100 {print}' data.txt

# Cut by delimiter
echo "name:value:extra" | cut -d: -f2     # "value"

# Get the Nth field from each line
docker images | awk 'NR>1 {print $1 ":" $2, $7}'   # repo:tag, size
```

### `xargs` patterns

```bash
# Run a command for each line of input
git diff --name-only | xargs wc -l

# Parallel execution
find /tmp -name "*.log" -print0 | xargs -0 -P 4 gzip

# With placeholder
echo "a b c" | xargs -I{} echo "Processing: {}"

# Limit batch size
cat urls.txt | xargs -n 5 curl -sf
```

### Writing temp scripts for complex transforms

When parsing gets complex, write a script to `/tmp/` instead of fighting
with shell quoting:

```bash
cat > /tmp/parse_output.py << 'PYEOF'
import sys
import json

data = json.load(sys.stdin)
for item in data.get("results", []):
    if item.get("score", 0) > 0.8:
        print(f"{item['name']}: {item['score']:.2f}")
PYEOF

cat results.json | python3 /tmp/parse_output.py
```

**Rule:** Any parsing logic over 3 pipes or any logic involving conditionals
should be a temp script. Shell one-liners break silently with quoting issues.

### Sorting and deduplication

```bash
# Sort lines alphabetically
sort output.txt

# Sort numerically by 2nd column
sort -k2 -n output.txt

# Unique lines (input must be sorted)
sort output.txt | uniq

# Unique with count
sort output.txt | uniq -c | sort -rn

# Top 10 most frequent
sort output.txt | uniq -c | sort -rn | head -10
```

---

## Playbook 8 — Timeout and Background Patterns

### Using the `timeout` parameter

The Bash tool accepts a `timeout` parameter in milliseconds (max 600000 = 10 min).

```
Default: 120000ms (2 minutes)
Max: 600000ms (10 minutes)
```

Use longer timeouts for:
- `npm install` on large projects → `timeout: 300000`
- Full test suites → `timeout: 300000`
- Docker builds → `timeout: 600000`
- Database migrations → `timeout: 300000`

Use shorter timeouts for:
- Quick checks (`git status`, `ls`) → default is fine
- Healthchecks → `timeout: 10000`

### `run_in_background` patterns

```bash
# Start a dev server — set run_in_background: true
npm run dev

# Start a database — set run_in_background: true
docker compose up postgres

# Run a long build you'll check later — set run_in_background: true
npm run build
```

After starting a background process, you will be **notified when it completes**.
Do NOT poll for completion. Do NOT use `sleep` loops.

### What NOT to do

```bash
# WRONG — sleep loop to wait for background process
npm run dev &
while ! curl -sf http://localhost:8080; do sleep 2; done

# RIGHT — use run_in_background for the server, then a separate call to check
# Call 1 (run_in_background: true): npm run dev
# Call 2 (timeout: 30000): curl -sf http://localhost:8080
```

```bash
# WRONG — sleep between commands that can run immediately
npm run lint
sleep 5       # pointless delay
npm run test

# RIGHT — just chain them
npm run lint && npm run test
```

```bash
# WRONG — sleep as a rate limiter in a retry loop
for i in 1 2 3; do curl http://api.example.com && break; sleep 10; done

# RIGHT — diagnose why it's failing; don't retry blindly
curl -v http://api.example.com 2>&1
```

### Monitoring long-running processes

Use the **Monitor** tool (not Bash) to stream stdout from a background process.
Each stdout line becomes a notification. This is better than `tail -f` in Bash
because it integrates with the tool notification system.

For one-shot "wait until done" tasks, `run_in_background` is sufficient — you
will be notified on completion without any polling.

---

## Anti-Patterns — What Bash Is NOT For

These operations have dedicated tools that are faster, safer, and produce
better-formatted output. Using Bash for them is always wrong.

| Do NOT use Bash for | Use this instead | Why |
|---------------------|------------------|-----|
| `find . -name "*.ts"` | **Glob** tool | Glob is optimized, sorted by mtime, no permission errors |
| `grep -r "pattern" src/` | **Grep** tool | Grep tool handles permissions, output formatting, limits |
| `rg "pattern" src/` | **Grep** tool | Same — Grep wraps rg with correct access controls |
| `cat file.ts` | **Read** tool | Read shows line numbers, handles encoding, reads images/PDFs |
| `head -50 file.ts` | **Read** with `limit: 50` | Read tool supports offset + limit natively |
| `tail -20 file.ts` | **Read** with `offset` | Calculate offset from file length |
| `sed -i 's/old/new/' file` | **Edit** tool | Edit is atomic, shows diffs, prevents partial writes |
| `awk '{sub(/old/,"new")}1' f` | **Edit** tool | Same — Edit is the file mutation tool |
| `echo "content" > file` | **Write** tool | Write tracks file creation, prevents accidental overwrites |
| `vim file` / `nano file` | **Edit** or **Write** | Interactive editors are not supported |
| `less file` | **Read** tool | Interactive pagers are not supported |
| `git add -i` | `git add <files>` | Interactive mode is not supported |
| `git rebase -i` | Non-interactive rebase | Interactive mode is not supported |
| `sleep 30` (first command) | Nothing — just run the next command | Sleep as first command with N >= 2 is blocked |

### The one exception

When a Bash command produces **output you need to parse** (like `npm run test`
or `docker compose ps`), you may pipe through `grep`, `awk`, `head`, etc.
within that same Bash call. The anti-pattern is using Bash to read/search
files that exist on disk — not filtering runtime command output.

```bash
# OK — filtering command output (not a file read)
npm run test 2>&1 | tail -20
docker compose ps | grep "running"
git log --oneline -5

# NOT OK — reading/searching files (use dedicated tools)
cat src/store/loanStore.tsx    # use Read tool
grep -r "useState" src/        # use Grep tool
```

---

## Quick Reference — Common Recipes

### Project bootstrap

```bash
# Node project
npm ci && npm run build && npm run test

# Python project
python3 -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]" && pytest

# Rust project
cargo build && cargo test
```

### Database operations

```bash
# Run Alembic migration
cd /path/to/project && alembic upgrade head

# Check current migration
alembic current

# Generate a new migration
alembic revision --autogenerate -m "add users table"

# Prisma
npx prisma migrate dev
npx prisma generate

# Supabase
npx supabase db push
npx supabase db diff
```

### System diagnostics

```bash
# Disk space
df -h .

# Memory usage
top -l 1 | head -10          # macOS
free -h                       # Linux

# Node version
node -v && npm -v

# Python version
python3 --version && pip --version

# Check if a CLI tool exists
command -v docker && echo "docker available" || echo "docker NOT found"
command -v aws && echo "aws-cli available" || echo "aws-cli NOT found"
```

### Cleanup operations

```bash
# Remove node_modules and reinstall
rm -rf node_modules && npm ci

# Clear npm cache
npm cache clean --force

# Remove Python cache
find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null
find . -name "*.pyc" -delete 2>/dev/null

# Remove build artifacts
rm -rf dist build .next .nuxt
```

---

## Output Format

When reporting Bash command results, use this structure:

```
## Command: `npm run test`
Exit code: 0 (success)

Key output:
- 47 tests passed, 0 failed
- Duration: 3.2s
- Coverage: 87% statements

No action needed.
```

For failures:

```
## Command: `npm run build`
Exit code: 1 (FAILED)

Error summary:
- TS2345: Argument of type 'string' is not assignable to parameter of type 'number'
  at src/lib/computations.ts:42

Root cause: `calculateEMI` expects numeric rate but receives string from form input.
Fix: Add `Number()` coercion at the call site.
```

Always report: command run, exit code, key output or error, and recommended
next step. Never dump raw stdout/stderr without summarizing — extract the
signal from the noise.
