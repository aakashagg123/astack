---
name: advanced-grep
description: >
  Enhanced codebase search and file reading for developers. Invoke this skill
  whenever you need to: search across multiple files with surrounding context,
  find all usages or references to a symbol/function/variable/class across the
  codebase, read large files that exceed normal limits or return errors, perform
  fuzzy or approximate pattern matching (case-insensitive, partial, regex),
  search inside structured formats (JSON, CSV, YAML) by key or field value,
  correlate definitions with all usages across files, or when the native Grep
  or Read tools return truncated, incomplete, empty, or unexpected results.
  Trigger phrases: "search the codebase for", "find all usages/references of",
  "where is X defined or used", "grep for X across files", "read this large
  file", "find all imports of", "search inside JSON/YAML/CSV", "cross-file
  search", "track down where X comes from", "which files use X".
  Always prefer this skill over native tools for any multi-file or large-file
  operation — even when the native tools seem like they should work.
---

# Advanced Search & Read

The core principle: **`ripgrep` (`rg`) and `fd` via Bash are more reliable and
more powerful than the native Grep/Read tools for anything non-trivial.** Use
them by default for multi-file searches, large files, format-aware searches, and
symbol correlation. Native tools are fine for quick single-file reads or
single-shot pattern checks — but for scale or complexity, use this playbook.

---

## Tool Selection Matrix

| Task | Tool |
|------|------|
| Single known file, < 300 lines | `Read` tool |
| Quick pattern, small directory | `Grep` tool |
| Multi-file search, need context | `Bash` + `rg -C N` |
| Large file (> 500 lines) | `Bash` chunked read or `rg` |
| Fuzzy / approximate match | `Bash` + `rg` with regex |
| Symbol: definition → all usages | Two-pass `rg` (see §4) |
| JSON search by key or value | `Bash` + `jq` |
| CSV search by column | `Bash` + Python csv module |
| YAML search by key | `Bash` + Python yaml module |
| Find files by name/type/recency | `Bash` + `fd` |

---

## Playbook 1 — Multi-file Pattern Search with Context

```bash
# Search with 3 lines of context, scoped to a directory
rg "PATTERN" --context 3 src/

# Scope to specific file type
rg "PATTERN" --type ts src/
rg "PATTERN" --glob '*.tsx' src/

# Show count summary first, then drill in
rg "PATTERN" --stats src/

# Show the enclosing function/class (more before, less after)
rg "PATTERN" -B 10 -A 3 src/components/
```

**Multi-pattern search (OR):** When a library has multiple usage forms, search
all of them in one pass rather than running separate queries and potentially
missing one pattern:

```bash
# Supabase: covers BOTH auth methods AND database query client (.from())
# Pattern 1: by variable name (common case)
rg "(supabase\.(auth|from|rpc|storage))" src/
# Pattern 2: by API signature, regardless of variable name (catches aliased clients)
rg '\.from\("[a-z_]' src/      # e.g. client.from("table"), sb.from("profiles")

# API calls: fetch + axios + custom client in one sweep
rg "(fetch\(|axios\.|apiClient\.|httpClient\.)" src/

# React hook variants
rg "(useState|useReducer|useContext)\(" src/
```

**Critical: always search by API signature, not just variable name.** The most
common miss is a library client assigned to a non-standard variable name. For
Supabase, `.from("tableName"` is a unique signature that identifies DB queries
regardless of what the client variable is called (`supabase`, `supabaseClient`,
`client`, `sb`, etc.). Similarly `supabase.auth.*` vs `sb.auth.*` — if you
don't know the import alias, search both ways.

Rule: when a search returns fewer results than expected, check whether the
client was imported under a different name. Use `rg "from.*supabase"` to find
the import statement and see the actual alias used.

If `rg` itself isn't available, fall back to:
```bash
grep -r --include="*.ts" -n -C 3 "PATTERN" src/
```

---

## Playbook 2 — Reading Large Files

When `Read` truncates or errors, do this in two steps:

```bash
# Step 1: Get total line count
wc -l path/to/file

# Step 2: Read in windows
sed -n '1,200p' path/to/file       # first 200 lines
sed -n '201,400p' path/to/file     # next 200
```

For CSV/TSV — always check schema before reading data:
```bash
head -3 file.csv        # header + 2 sample rows
wc -l file.csv          # total row count
```

For JSON — check top-level structure first:
```bash
jq 'keys' file.json                    # top-level keys
jq 'length' file.json                  # array length if array
jq '.[0]' file.json                    # first element if array
```

---

## Playbook 3 — Fuzzy and Approximate Matching

Native Grep is exact. These patterns give you fuzziness:

```bash
# Case insensitive
rg -i "pattern" src/

# Optional character (colour / color)
rg "colou?r" src/

# Multiple synonyms / naming conventions
rg "(get|fetch|load)User" src/
rg "(initialise|initialize|init)" src/

# Whole-word match only (no substring false positives)
rg --word-regexp "Auth" src/

# Literal string (disable regex, useful when pattern has dots/parens)
rg -F "auth.config()" src/

# Allow any single character between two parts
rg "use.Auth" src/    # matches useAuth, use_Auth, etc.
```

---

## Playbook 4 — Cross-File Correlation (Definition → All Usages)

This is a three-pass workflow. Run all three, then synthesize.

**Pass 1: Find the definition**
```bash
rg "^(export )?(function|const|class|interface|type) MySymbol\b" src/
```

**Pass 2: Find all call sites / usages**
```bash
rg "\bMySymbol\b" src/ --stats
```

**Pass 3: Which files depend on it (for dependency map)**
```bash
rg "\bMySymbol\b" src/ -l          # files only
rg "import.*MySymbol" src/         # import statements specifically
```

For TypeScript module dependencies:
```bash
rg "from ['\"].*my-module['\"]" src/     # all importers of a module
rg "require\(['\"].*my-module" src/      # CommonJS importers
```

Synthesize into a dependency map (see Output Format §6).

---

## Playbook 5 — Format-Aware Search

### JSON — search by key or value

```bash
# Check if a key exists anywhere in the document
jq 'paths | select(. == ["targetKey"])' file.json

# Find all objects with a specific key
jq '.. | objects | select(has("targetKey"))' file.json

# Find by value
jq '.. | objects | select(.status == "active")' file.json

# Extract specific field from all array elements
jq '.[].name' file.json
```

When the file is very large and `jq` is slow, use `rg` to narrow scope first:
```bash
rg '"targetKey"' file.json -n      # find line numbers first
```

### CSV — search, filter, and aggregate by column

For schema + sample rows (simple):
```bash
head -3 file.csv        # header + 2 data rows
wc -l file.csv          # total row count
```

For filter by column value (write to /tmp/csv_search.py, then run):
```python
import csv
with open('file.csv') as f:
    reader = csv.DictReader(f)
    print("Columns:", reader.fieldnames)
    matches = [r for r in reader if r.get('ColumnName', '').lower() == 'value']
    for m in matches[:20]:
        print(m)
```

For aggregation / pivot (count by category, sorted):
```python
import csv
from collections import Counter

with open('file.csv') as f:
    reader = csv.DictReader(f)
    # Filter then count sub-categories
    counter = Counter()
    total = 0
    for row in reader:
        if row.get('Segment', '') == 'Target Segment':
            counter[row.get('Sub-segment', 'Unknown')] += 1
            total += 1

print(f"Total matching rows: {total}")
print("\nBreakdown by Sub-segment (desc):")
for name, count in counter.most_common():
    print(f"  {name}: {count}")
```

Always write multi-step scripts to `/tmp/csv_script.py` and run with
`python3 /tmp/csv_script.py` rather than trying to inline them in a
`python3 -c` one-liner — complex logic breaks in single-quoted shells.

### YAML — search by key

```python
import yaml
with open('file.yaml') as f:
    data = yaml.safe_load(f)

# Recursive key finder
def find_key(obj, target, path=''):
    if isinstance(obj, dict):
        for k, v in obj.items():
            if k == target:
                print(f"{path}.{k} = {v}")
            find_key(v, target, f"{path}.{k}")
    elif isinstance(obj, list):
        for i, item in enumerate(obj):
            find_key(item, target, f"{path}[{i}]")

find_key(data, 'targetKey')
```

---

## Playbook 6 — Finding Files

```bash
# Find by name pattern
fd "component" src/ --type f

# Find by extension
fd -e tsx src/
fd -e py --type f .

# Find recently modified (last 24h)
fd --changed-within 1d src/

# Find and search inside results
fd -e ts src/ | xargs rg "useEffect"

# Find by size (> 100KB)
fd --size +100k .
```

---

## When Native Tools Fail — Diagnostics

If `Grep` returns 0 results but you're confident matches exist:
- File is gitignored → `rg --no-ignore PATTERN`
- Special characters in pattern → `rg -F "literal.string"`
- Encoding issue → `rg --encoding utf-8 PATTERN`
- Wrong directory scope → widen path or use `.` for cwd

If `Read` fails or truncates:
- File too large → use `wc -l` first, then `Read` with `offset` + `limit` params
- Binary file → use `rg --text` or `file` command to check type
- Permission denied → `ls -la path/to/file`

If results seem incomplete:
- Add `--stats` to `rg` to see total match count vs displayed
- Use `-l` (files only) then drill into specific files
- Use `--max-count 5` to cap per-file results and widen coverage

---

## Output Format

Always present search results in this structured format:

```
## Search: `PATTERN` in `PATH`
Matches: N results across M files

### [src/path/to/file.ts:42](src/path/to/file.ts#L42)
```
[context before]
→ MATCHING LINE
[context after]
```

### [src/other/file.ts:87](src/other/file.ts#L87)
...
```

For cross-file correlation, add a **Dependency Map**:

```
## Symbol: `MySymbol`
- Defined in: [src/utils/auth.ts:15](src/utils/auth.ts#L15)
- Imported by 4 files:
  - [src/pages/Login.tsx:3](src/pages/Login.tsx#L3)
  - [src/hooks/useAuth.ts:1](src/hooks/useAuth.ts#L1)
  - [src/components/Guard.tsx:8](src/components/Guard.tsx#L8)
  - [src/api/client.ts:22](src/api/client.ts#L22)
```

Always use clickable markdown links (`[file.ts:42](file.ts#L42)`) for every
file reference — these open directly in the editor.
