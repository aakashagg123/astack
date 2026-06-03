---
name: advanced-glob
description: >
  Enhanced file finding and pattern matching for developers. Invoke this skill
  whenever you need to: find files by complex name patterns, locate files by
  extension across deep directory trees, find recently modified files, discover
  project structure and layout, find files matching multiple criteria (name +
  location + type), locate config files across monorepos, find test files
  corresponding to source files, map out directory structures, or when the
  native Glob tool returns unexpected results or too many matches.
  Trigger phrases: "find all files matching", "where are the test files",
  "show me the project structure", "find config files", "what files changed
  recently", "find all .tsx files in", "locate the schema files", "map the
  directory tree", "find files by extension", "show file layout".
  Always prefer this skill over raw find/ls for file discovery tasks.
---

# Advanced Glob — File Finding & Pattern Matching

The core principle: **the Glob tool is the primary file finder, `fd` via Bash
handles what Glob cannot, and `ls` is for quick one-directory peeks.** Never use
`find` or `ls -R` when Glob or `fd` can do the job. This skill covers file
*discovery* only — for content search use `advanced-grep`, for shell execution
use `advanced-bash`.

---

## Tool Selection Matrix

| Task | Tool | Why |
|------|------|-----|
| Files by extension in a subtree | `Glob` — `**/*.tsx` | Native, fast, sorted by mtime |
| Files by name fragment | `Glob` — `**/*Schema*` | Simple wildcard matching |
| Files in one directory | `Bash` + `ls` | Quick peek, no recursion needed |
| Files by size | `Bash` + `fd --size +100k` | Glob has no size filter |
| Files by modification time | `Bash` + `fd --changed-within 1d` | Glob has no date filter |
| Files by type (file/dir/symlink) | `Bash` + `fd --type d` | Glob returns files only |
| Files excluding a pattern | `Bash` + `fd -E node_modules` | Glob has no negation |
| Files with depth limit | `Bash` + `fd --max-depth 2` | Glob always recurses fully |
| Files matching multiple criteria | `Bash` + `fd` with combined flags | Compound filters |
| Verify a path exists | `Glob` with exact pattern | Fast existence check |
| Count files by type | `Bash` + `fd -e tsx \| wc -l` | Glob returns list, not count |

**Decision rule:** Start with Glob. If you need size, date, depth, negation, or
type filtering — switch to `fd`. If you just need one directory listing — use
`ls`.

---

## Playbook 1 — Basic Glob Patterns

### Extension matching

```
# All TypeScript files in the project
**/*.ts

# All React component files
**/*.tsx

# All JSON files under a specific directory
src/store/**/*.json

# All test files
**/*.test.ts
**/*.test.tsx
```

### Name fragment matching

```
# Files containing "Schema" in the name
**/*Schema*

# Files starting with "use" (hooks)
**/use*.ts
**/use*.tsx

# Files ending with "Store"
**/*Store.ts
```

### Brace expansion — multiple extensions or directories

```
# TypeScript AND TypeScript React files
**/*.{ts,tsx}

# Files in either src or lib directories
{src,lib}/**/*.ts

# Config files by multiple names
**/{tsconfig,vite.config,vitest.config}.*

# Test OR spec files
**/*.{test,spec}.{ts,tsx}
```

### Directory scoping with the `path` parameter

Always use the `path` parameter to narrow scope when you know the target area:

```
# Only in src/store/
pattern: **/*.ts
path: /absolute/path/to/src/store

# Only in components
pattern: **/*.tsx
path: /absolute/path/to/src/components
```

### Key Glob tool parameters

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `pattern` | The glob pattern (required) | `**/*.tsx` |
| `path` | Root directory to search from | `/Users/you/project/src` |

Results are sorted by modification time (most recent first) — useful for finding
what was recently touched without needing `fd --changed-within`.

---

## Playbook 2 — Finding Test & Spec Files

### Convention mapping: source to test

| Source file | Test file (convention 1) | Test file (convention 2) |
|-------------|-------------------------|-------------------------|
| `Button.tsx` | `Button.test.tsx` | `__tests__/Button.test.tsx` |
| `loanStore.ts` | `loanStore.test.ts` | `loanStore.spec.ts` |
| `computations.ts` | `computations.test.ts` | `computations.spec.ts` |

### Find all test files

```
# All test files in the project
**/*.test.{ts,tsx}

# All spec files
**/*.spec.{ts,tsx}

# Both conventions
**/*.{test,spec}.{ts,tsx}

# Jest __tests__ directories
**/__tests__/**/*.{ts,tsx}
```

### Find the test for a specific source file

Given `src/store/formSchemaStore.ts`, search for its test:

```
# Exact match
**/formSchemaStore.test.ts
**/formSchemaStore.spec.ts

# Also check for split test files (common in large projects)
**/formSchemaStore.*.test.ts

# Check __tests__ directory
**/__tests__/formSchemaStore*
```

### Find source files that lack tests

Two-step workflow:

1. Glob for all source files: `src/**/*.{ts,tsx}` (exclude test files mentally
   or via fd)
2. Glob for all test files: `**/*.test.{ts,tsx}`
3. Compare the lists — any source file without a corresponding test is a gap

For automated detection, use `fd` via Bash:

```bash
# All .ts files that are NOT test/spec files
fd -e ts -e tsx --exclude '*.test.*' --exclude '*.spec.*' --exclude '__tests__' src/
```

---

## Playbook 3 — Project Structure Mapping

### Understand a new codebase fast

Run these three globs in parallel to get the lay of the land:

```
# 1. Entry points and config
**/{index,main,App,app}.{ts,tsx,js,jsx}

# 2. Package/project definition files
**/package.json
**/tsconfig*.json
**/{vite,vitest,webpack,rollup,jest}.config.*

# 3. Store / state management files
**/*{Store,store,Context,context}*.{ts,tsx}
```

### Find all page/route components

```
# Next.js pages
**/pages/**/*.{tsx,jsx}
**/app/**/page.{tsx,jsx}

# React Router style (convention: src/pages/)
src/pages/**/*.tsx

# Named route files
**/*Page.tsx
**/*View.tsx
**/*Screen.tsx
```

### Find all hook files

```
**/use*.{ts,tsx}
**/hooks/**/*.{ts,tsx}
```

### Find all type definition files

```
**/*.d.ts
**/types/**/*.ts
**/types.ts
**/*Types.ts
**/interfaces/**/*.ts
```

### Map component hierarchy

```
# Top-level components
src/components/*.tsx

# All components recursively
src/components/**/*.tsx

# UI primitives (shadcn/radix pattern)
src/components/ui/**/*.tsx

# Feature-specific components
src/components/{los,cam,admin}/**/*.tsx
```

### Quick directory overview (use ls, not Glob)

```bash
# Top-level structure
ls src/

# One level deep in components
ls src/components/

# Two levels for a monorepo
ls packages/*/
```

---

## Playbook 4 — Monorepo Navigation

### Find all packages/workspaces

```
# Standard monorepo patterns
**/package.json

# Narrow to workspace roots (not nested node_modules)
packages/*/package.json
apps/*/package.json
libs/*/package.json
```

### Find shared configuration

```
# Root config that applies everywhere
tsconfig.json
.eslintrc*
.prettierrc*
jest.config.*
vitest.config.*

# Per-package overrides
**/tsconfig.json
**/.eslintrc*
```

### Cross-package dependency discovery

```bash
# Which packages have React as a dependency
fd package.json --max-depth 3 | xargs grep -l '"react"'

# Which packages have a specific internal dependency
fd package.json --max-depth 3 | xargs grep -l '"@myorg/shared"'
```

### Find shared utilities across packages

```
# Common shared module patterns
**/shared/**/*.ts
**/common/**/*.ts
**/utils/**/*.ts
**/lib/**/*.ts
```

---

## Playbook 5 — Advanced Criteria with `fd` (Bash)

`fd` is the power tool when Glob's pattern matching is insufficient. It handles
size, date, depth, type, and exclusion filters that Glob cannot express.

### By size

```bash
# Files larger than 100KB (find bloated files)
fd --size +100k --type f .

# Files larger than 1MB (likely binary or generated)
fd --size +1m --type f .

# Small files only (< 1KB — likely stubs or empty)
fd --size -1k --type f src/
```

### By modification time

```bash
# Changed in the last 24 hours
fd --changed-within 1d --type f src/

# Changed in the last hour (active development)
fd --changed-within 1h --type f src/

# Changed in the last week
fd --changed-within 7d --type f src/

# NOT changed in 30 days (stale files)
fd --changed-before 30d --type f src/
```

### By type (file, directory, symlink)

```bash
# Only directories
fd --type d src/

# Only symlinks
fd --type l .

# Only files (default, but explicit)
fd --type f src/
```

### Excluding patterns

```bash
# All TS files, excluding tests
fd -e ts -e tsx -E '*.test.*' -E '*.spec.*' -E '__tests__' src/

# All files excluding node_modules and dist
fd --type f -E node_modules -E dist .

# All JSON files excluding package-lock
fd -e json -E package-lock.json .
```

### Depth limiting

```bash
# Only top-level directories in src/
fd --max-depth 1 --type d . src/

# Files no deeper than 2 levels
fd --max-depth 2 --type f src/

# Files at exactly depth 3 (min + max)
fd --min-depth 3 --max-depth 3 --type f src/
```

### Combining criteria

```bash
# Large TypeScript files modified today
fd -e ts -e tsx --size +50k --changed-within 1d src/

# Recently modified config files
fd --changed-within 7d '(config|rc|settings)\.' .

# Empty directories (cleanup candidates)
fd --type d --type empty .

# Executable files
fd --type x .
```

### Limiting results

```bash
# Only first 20 results (quick peek)
fd -e tsx src/ | head -20

# Count without listing
fd -e tsx src/ | wc -l
```

---

## Playbook 6 — Combining Glob with Other Tools

The **find-filter-act** pipeline: Glob finds candidates, then other tools
refine and inspect.

### Pattern: Glob to find, Read to inspect

```
Step 1: Glob for **/*Store.ts (find all stores)
Step 2: Read the top 50 lines of each (understand exports)
Step 3: Report the store inventory
```

### Pattern: Glob to find, Grep to search within

```
Step 1: Glob for src/components/**/*.tsx (find all components)
Step 2: Grep for "useEffect" across those files (find side effects)
Step 3: Report which components have effects
```

### Pattern: Glob to verify, then act

Before editing a file, verify it exists and check for siblings:

```
Step 1: Glob for **/targetFile.ts (does it exist?)
Step 2: Glob for **/targetFile.* (any related files? tests? types?)
Step 3: Read the target file
Step 4: Edit with confidence
```

### Pattern: Glob for inventory, fd for filtering

```
Step 1: Glob for **/*.tsx in src/components/ (full inventory)
Step 2: fd -e tsx --size +10k src/components/ (which are large?)
Step 3: fd -e tsx --changed-within 1d src/components/ (which are active?)
Step 4: Prioritize review targets from the intersection
```

### Pattern: fd for candidates, rg for content filtering

```bash
# Find large recently-modified TypeScript files that import from a store
fd -e ts -e tsx --size +5k --changed-within 7d src/ \
  | xargs rg -l "from.*Store" 2>/dev/null
```

---

## Playbook 7 — Handling Large Result Sets

When Glob returns hundreds of files, narrow the scope before processing.

### Strategy 1: Tighten the path parameter

```
# Too broad — returns everything
pattern: **/*.tsx

# Better — scoped to feature area
pattern: **/*.tsx
path: /project/src/components/los

# Best — scoped to exact directory
pattern: *.tsx
path: /project/src/components/los/stages
```

### Strategy 2: Make the pattern more specific

```
# Too many results
**/*.ts

# Better — only store files
**/*Store.ts
**/*store*.ts

# Better — only hook files
**/use*.ts
```

### Strategy 3: Use fd with --max-results

```bash
# Cap at 50 results
fd -e tsx --max-results 50 src/

# Cap and sort by size (find the biggest)
fd -e tsx src/ --exec ls -la {} \; | sort -k5 -n -r | head -20
```

### Strategy 4: Use fd with --max-depth

```bash
# Only 2 levels deep — skip deeply nested files
fd -e tsx --max-depth 2 src/

# Top-level directory survey
fd --max-depth 1 --type d src/
```

### Strategy 5: Two-pass approach

```
Pass 1: Glob for **/*.tsx with path scoped to src/
        → Too many? Note the top-level directories from results.

Pass 2: Glob separately for each relevant subdirectory:
        src/components/**/*.tsx
        src/pages/**/*.tsx
        src/hooks/**/*.tsx
        → Manageable result sets per domain.
```

---

## Anti-Patterns

| Do NOT do this | Do this instead | Why |
|----------------|-----------------|-----|
| `find . -name "*.tsx"` via Bash | `Glob` with `**/*.tsx` | Glob is the native tool; find is slower and less integrated |
| `ls -R src/` via Bash | `Glob` with `src/**/*` or `fd . src/` | Recursive ls is unformatted and hard to parse |
| `Glob` with `**/*` (no extension) | `fd --type f .` or scope tighter | Returns every file — too many results, wastes context |
| Assume a path exists from memory | Glob for `**/fileName*` first | Paths drift between sessions; verify before acting |
| Use Glob for content search | Use `Grep` tool or `rg` via Bash | Glob finds files by name; Grep/rg search file contents |
| Hardcode absolute paths | Glob to discover, then use the returned path | Avoids typos and stale path references |
| `fd` without `-E node_modules` | Always add `-E node_modules -E .git` | node_modules has thousands of files; .git has internal objects |
| Chain multiple `ls` calls to explore | One `fd --max-depth 2 --type d .` | Single command gives full picture |
| Glob the same pattern repeatedly | Cache results mentally; re-glob only if files may have changed | Each Glob is a tool call — avoid redundant calls |

---

## Codebase-Specific Patterns

Pre-build glob patterns for your project's file layout by adapting these examples to your directory structure.

### Schema / config files

```
# All JSON schema files under a specific directory
pattern: *_schema.json
path: src/store/schemas

# All config files in the project
pattern: **/*.config.{ts,js,json}
path: .
```

### Workflow / process files

```
# BPMN workflow files
pattern: **/*.bpmn20.xml
path: src/bpmn

# DMN decision table files
pattern: **/*.dmn
path: src/bpmn
```

### Component files

```
# All components under a feature area
pattern: **/*.tsx
path: src/components/feature-name

# All components in the project
pattern: **/*.{tsx,jsx}
path: src/components
```

### Store / state files

```
# All store files
pattern: **/*.ts
path: src/store
```

### Test files

```
# All test files in the project
pattern: **/*.test.{ts,tsx}
path: src

# Test files matching a naming convention (e.g. .t24.test.tsx)
pattern: **/*.t*.test.tsx
path: src
```

### Skill files

```
# All skills in the project
pattern: **/skills/*/SKILL.md

# Global skills
pattern: */SKILL.md
path: ~/.claude/skills
```

### Type definition files

```
# Types under a feature area
pattern: **/*.ts
path: src/types/feature-name

# All types
pattern: **/*.ts
path: src/types
```

### Spec and documentation files

```
# All spec markdown files
pattern: *.md
path: specs

# HTML spec companion files
pattern: *.html
path: specs
```

---

## Output Format

Present file discovery results in this structured format:

```
## Files: `PATTERN` in `PATH`
Found: N files

### By directory
- `src/store/` — 12 files
  - formSchemaStore.ts (14KB, modified 2h ago)
  - loanStore.tsx (28KB, modified 1d ago)
  - ...
- `src/components/` — 34 files
  - ...

### Notable
- Largest: `src/store/loanStore.tsx` (28KB)
- Most recent: `src/store/cf/cfSchemaData.ts` (2h ago)
- Missing expected: No test file for `camSchemaStore.ts`
```

For project structure mapping, use a tree format:

```
## Project Structure: `src/`

src/
  components/
    admin/        — 8 files (panels, config UIs)
    cam/
      cf/         — 17 files (CF CAM renderers)
      tl/         — 0 files (future)
    los/          — 12 files (stage components)
    ui/           — 24 files (shadcn primitives)
  contexts/       — 3 files (Role, Auth, Cam)
  hooks/          — 7 files (useAbac, useQueryDirectedAccess, ...)
  lib/            — 15 files (engines, utilities, registries)
  pages/          — 6 files (routes)
  store/
    cf/           — 4 files (CF schema data + schemas/)
    tl/           — 0 files (future)
  types/
    cf/           — 2 files (CF-specific types)
```

For test coverage mapping:

```
## Test Coverage: `src/store/`

| Source file | Test file | Status |
|-------------|-----------|--------|
| formSchemaStore.ts | formSchemaStore.version.test.ts | covered |
| loanStore.tsx | loanStore.helpers.test.ts | partial |
| camSchemaStore.ts | — | missing |
| abacPolicyStore.ts | abacPolicyStore.test.ts | covered |
```

---

## Trio Reference

This skill is part of a trio. Use the right one:

| Skill | Domain | When |
|-------|--------|------|
| **advanced-glob** (this) | File finding by name/path/pattern | "Where are the files?" |
| **advanced-grep** | Content search within files | "What's inside the files?" |
| **advanced-bash** | Shell execution and scripting | "Run this command" |

If you need to find files AND search their contents, start with advanced-glob to
locate candidates, then switch to advanced-grep to search within them.
