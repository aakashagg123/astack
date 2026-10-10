---
name: advanced-search
description: Find and read things across a codebase or document set, and run multi-step shell work. Covers three jobs, each with its own playbook — content search (symbols, usages, JSON/CSV/YAML, large files), file discovery (by name, extension, recency, project layout), and shell execution (pipelines, builds, processes, Docker, migrations). Use when a task spans many files, the native Grep/Glob/Read tools truncate or return nothing, or a shell task needs more than a one-liner. Do not use for a single known file or a quick one-pattern check.
---

# Advanced Search

One skill, three playbooks. Pick the job, then read only that reference.

| Job | Signal | Read |
|---|---|---|
| Search inside files | "find where X is used", "which files reference", "search JSON/YAML/CSV", "read this large file" | `references/grep.md` |
| Find files | "find files named", "list all .tsx", "where is the config", "what changed recently", "map the project" | `references/glob.md` |
| Run shell work | "build and test", "kill the process on port", "docker compose", "run the migration", "chain these commands" | `references/bash.md` |

## Procedure

1. Classify the job from the table.
2. Read the one matching reference file. Do not read the other two.
3. Follow its playbook.
4. If the task needs two jobs, run them in order: find files, search inside them, then run commands.
5. If two jobs could both fit, state both in one line each and ask one question.

## Default rule

Use the dedicated tool (`Read`, `Grep`, `Glob`) for a single file or a quick check. Use the playbook for scale, context lines, format-aware search, or when the native tool fails.

## Confidence

End substantial work with: `Confidence: X% — [what was covered]. [Gaps: ignored dirs, skipped binaries, unmapped aliases.]`
