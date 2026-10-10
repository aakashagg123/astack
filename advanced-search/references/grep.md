
# Advanced Grep

Core principle: **`ripgrep` (`rg`) via Bash is faster and more accurate than the native Grep for anything non-trivial.** Use the native tool for a single known file; use `rg` for scale, context, and format-awareness.


## Tool selection

| Task | Tool |
|---|---|
| Single file, < 300 lines | native `Read` |
| Quick pattern, small dir | native `Grep` |
| Multi-file, need context | `rg -C N` via Bash |
| File > 500 lines | `wc -l` + windowed `sed` |
| Fuzzy or regex | `rg` with the right flag |
| Definition → all usages | Two-pass `rg` (Playbook 4) |
| JSON by key/value | `jq` via Bash |
| CSV by column | Python `csv` module |
| YAML by key | Python `yaml` module |
| Files by name | `glob.md` |


## Playbook 1 — Multi-file search with context

```bash
rg "PATTERN" --context 3 src/
rg "PATTERN" --type ts src/
rg "PATTERN" --glob '*.tsx' src/
rg "PATTERN" --stats src/
rg "PATTERN" -B 10 -A 3 src/components/
```

**Multi-pattern in one pass** — do not run four separate greps for four variants:

```bash
rg "(supabase\.(auth|from|rpc|storage))" src/
rg '\.from\("[a-z_]' src/
rg "(fetch\(|axios\.|apiClient\.|httpClient\.)" src/
rg "(useState|useReducer|useContext)\(" src/
```

**Rule: search by API signature, not just variable name.** The most common miss is a client aliased under a non-standard name. For Supabase, `.from("tableName"` catches DB queries regardless of whether the client is `supabase`, `sb`, `client`, or `db`. When results seem thin, find the import alias first:

```bash
rg "from.*supabase" src/
```

Fallback if `rg` is unavailable:
```bash
grep -r --include="*.ts" -n -C 3 "PATTERN" src/
```


## Playbook 2 — Read large files

When `Read` truncates:

```bash
wc -l path/to/file
sed -n '1,200p' path/to/file
sed -n '201,400p' path/to/file
```

CSV / TSV — check schema first:
```bash
head -3 file.csv
wc -l file.csv
```

JSON — check top-level shape first:
```bash
jq 'keys' file.json
jq 'length' file.json
jq '.[0]' file.json
```


## Playbook 3 — Fuzzy matching

```bash
rg -i "pattern" src/
rg "colou?r" src/
rg "(get|fetch|load)User" src/
rg "(initialise|initialize|init)" src/
rg --word-regexp "Auth" src/
rg -F "auth.config()" src/
rg "use.Auth" src/
```


## Playbook 4 — Definition → all usages (three passes)

```bash
# Pass 1: definition
rg "^(export )?(function|const|class|interface|type) MySymbol\b" src/

# Pass 2: call sites
rg "\bMySymbol\b" src/ --stats

# Pass 3: importers
rg "\bMySymbol\b" src/ -l
rg "import.*MySymbol" src/
rg "from ['\"].*my-module['\"]" src/
rg "require\(['\"].*my-module" src/
```

Synthesize into the Dependency Map (see Output).


## Playbook 5 — Format-aware search

### JSON

```bash
jq 'paths | select(. == ["targetKey"])' file.json
jq '.. | objects | select(has("targetKey"))' file.json
jq '.. | objects | select(.status == "active")' file.json
jq '.[].name' file.json
```

For huge files, narrow with `rg` first:
```bash
rg '"targetKey"' file.json -n
```

### CSV

Simple:
```bash
head -3 file.csv
wc -l file.csv
```

Filter by column value — write to `/tmp/csv_search.py`, then run:
```python
import csv
with open('file.csv') as f:
    reader = csv.DictReader(f)
    print("Columns:", reader.fieldnames)
    matches = [r for r in reader if r.get('ColumnName', '').lower() == 'value']
    for m in matches[:20]:
        print(m)
```

Aggregate (count by category):
```python
import csv
from collections import Counter

with open('file.csv') as f:
    reader = csv.DictReader(f)
    counter = Counter()
    total = 0
    for row in reader:
        if row.get('Segment') == 'Target':
            counter[row.get('Sub-segment', 'Unknown')] += 1
            total += 1

print(f"Total: {total}")
for name, count in counter.most_common():
    print(f"  {name}: {count}")
```

Always write multi-step scripts to `/tmp/` and run them; `python3 -c` one-liners break in single-quoted shells.

### YAML

```python
import yaml
with open('file.yaml') as f:
    data = yaml.safe_load(f)

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


## Diagnostics when native tools fail

`Grep` returns 0 but you expect matches:
- Gitignored → `rg --no-ignore PATTERN`
- Special chars → `rg -F "literal.string"`
- Encoding → `rg --encoding utf-8 PATTERN`
- Wrong scope → widen the path

`Read` truncates or errors:
- Large file → `wc -l`, then `Read` with `offset` + `limit`
- Binary → `file` to confirm type, then `rg --text`
- Permission → `ls -la path`

Results look incomplete:
- `--stats` shows total matches vs displayed
- `-l` to list files, then drill in
- `--max-count 5` to widen coverage per file


## Output format

```
## Search: `PATTERN` in `PATH`
Matches: N results across M files

### [src/path/to/file.ts:42](src/path/to/file.ts#L42)
```
[context before]
→ MATCHING LINE
[context after]
```
```

For cross-file correlation, add:

```
## Symbol: `MySymbol`
- Defined in: [src/utils/auth.ts:15](src/utils/auth.ts#L15)
- Imported by 4 files:
  - [src/pages/Login.tsx:3](src/pages/Login.tsx#L3)
  - [src/hooks/useAuth.ts:1](src/hooks/useAuth.ts#L1)
  - [src/components/Guard.tsx:8](src/components/Guard.tsx#L8)
  - [src/api/client.ts:22](src/api/client.ts#L22)
```

Always use clickable `[file:line](file#Lline)` links.


## Confidence

Close every substantial search with:
`Search confidence: X% — [N] matches across [M] files. [Gaps if any: gitignored dirs, binary files skipped, alias not fully mapped.]`
