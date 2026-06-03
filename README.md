# astack

**A Claude Code skill stack for builders who ship with intent.**

---

Most AI-assisted development fails the same way. Not because the code is wrong — because the *problem* was wrong. The spec was vague. The requirements weren't testable. The implementation plan ignored what was already in the codebase. The PR shipped the right syntax but the wrong behaviour.

astack is a set of Claude Code slash commands that close those gaps. It covers the full build cycle — from structured problem discovery through spec writing, test-driven implementation, security review, and cycle closure. Each skill is a focused tool. The `/astack` router dispatches any task to the right one automatically.

---

## What you get

**28 slash commands across 5 domains.**

### Product pipeline

| Skill | What it does |
| --- | --- |
| `/astack` | Routes any task to the right skill — the entry point for the whole stack |
| `/problem-discovery` | Six-phase structured discovery before any spec or code is written |
| `/preflight` | Validates that the environment is ready before any non-trivial task begins |
| `/prompt-builder` | Engineers a reusable, cold-hand-test-passing prompt from a vague brief |
| `/spec-creator` | Produces a complete spec with acceptance criteria, decisions register, and 1:1 test cases |
| `/doc-coauthoring` | Co-authors structured documents — PRDs, RFCs, design docs |

### Engineering pipeline

| Skill | What it does |
| --- | --- |
| `/tech-development-plan` | Sequences implementation from stories into a phased, dependency-ordered plan |
| `/10x-ai-engineer` | Full pipeline: brief → spec → stories → plan → TDD → review → sync |
| `/secure-coding-practices` | Security posture check before any auth, API key, or access-control change |
| `/red-green-tdd` | Strict TDD: failing test first, minimal pass, then refactor |
| `/evaluate-approach` | Scores competing implementation approaches against tradeoffs before committing |
| `/pr-review` | Reviews code against the original spec and acceptance criteria — not just style |
| `/context-sync` | Closes a dev cycle and updates the project's working memory with learnings |
| `/meta-learn` | Synthesises patterns and learnings across multiple sessions or projects |

### Skill stack management

| Skill | What it does |
| --- | --- |
| `/skill-creator` | Creates, edits, or improves any SKILL.md in the stack |
| `/autoresearch` | Benchmarks a skill against real tasks and improves it through evaluation loops |

### Research and search

| Skill | What it does |
| --- | --- |
| `/data-analysis` | Analyses and visualises structured datasets |
| `/advanced-search` | Routes search tasks to the right underlying tool |
| `/advanced-grep` | Multi-file content search, symbol correlation, and format-aware search |
| `/advanced-glob` | File discovery by name, pattern, and project structure |
| `/advanced-bash` | Complex shell pipelines, build/test/lint, process management |

### Document and artifact tools

| Skill | What it does |
| --- | --- |
| `/pdf` | PDF reading, extraction, and analysis |
| `/xlsx` | Spreadsheet input and analysis |
| `/pptx` | PowerPoint file tasks |
| `/docx` | Word document tasks |
| `/web-artifacts-builder` | Builds multi-component HTML/CSS/JS artifacts |
| `/mcp-builder` | Creates or improves an MCP server |
| `/brand-guidelines` | Applies Anthropic brand colours and typography to any artifact |

---

## Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) · [Git](https://git-scm.com/)

### For your user account (30 seconds)

Open Claude Code and run:

```sh
git clone https://github.com/aakashagg123/astack.git ~/.claude/skills/astack
```

All 28 slash commands are immediately available. No setup script. No config file. No dependencies.

### Add to your project

```sh
cp -Rf ~/.claude/skills/astack .claude/skills/astack
```

Commit `.claude/skills/astack/` to your repo. Anyone who clones the repo gets the full stack.

---

## Quick start

```sh
/astack list              → see the full skill map
/problem-discovery        → structure any vague requirement before writing a spec
/preflight                → validate your environment before a complex task
/10x-ai-engineer          → full end-to-end feature build
/pr-review                → review a PR against requirements, not just lint
```

---

## Philosophy

Read [ETHOS.md](ETHOS.md) for the reasoning behind the stack's structure — what it optimises for, what it deliberately ignores, and where it fits in a build workflow.

---

## License

MIT — see [LICENSE](LICENSE).
