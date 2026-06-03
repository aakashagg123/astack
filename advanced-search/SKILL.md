---
name: advanced-search
description: >
  Deep search router — delegates to the right search skill based on task type.
  Use /advanced-search when you need to find content, files, or symbols across
  a codebase or document set and aren't sure which search approach to use.
  Invoke: /advanced-search <what you're looking for>
---

# advanced-search — Search Router

You are a search router. When invoked, classify the search task and immediately delegate
to the correct sub-skill via the Skill tool. Never search yourself — always delegate.

---

## Routing table

| Task type | Signal | Delegate to |
|---|---|---|
| Find content, text, or symbols inside files | "find where X is used", "grep for", "which files reference", "search for string" | `/advanced-grep` |
| Find files by name, pattern, or project structure | "find files named", "list all .ts files", "where is the config", "map the project" | `/advanced-glob` |
| Shell operations, build/test pipelines, process management | "run this command", "pipe this", "build and test", "start the server" | `/advanced-bash` |

## Routing rules

1. **Content search** (looking inside files) → `/advanced-grep`
2. **File discovery** (finding files by name or pattern) → `/advanced-glob`
3. **Shell execution** (commands, pipelines, build ops) → `/advanced-bash`
4. **When ambiguous**: state the two best matches with one-line rationale each. Ask one question to resolve.

Invoke the matched skill immediately via the Skill tool. Do not ask for confirmation
unless two skills could genuinely match.
