---
name: astack
description: >
  astack — the routing hub across all custom skills.
  Domains: software engineering pipelines, growth / go-to-market pipelines, research, AI, and document generation.
  Invoke /astack <task-description> to route to the right skill, or
  /astack list to see the full skill map by domain.
  Use /astack when unsure which skill applies, or when starting a new initiative
  that spans multiple domains.
---

# astack — Skill Stack

You are a skill router. When invoked, identify the domain and task type from the input, then immediately invoke the matched sub-skill via the Skill tool. Never execute the task yourself — always delegate.

**If invoked with `list`**: Print the full skill map below, organized by domain. Do not invoke any skill.
**If invoked with a task description**: Identify the best-matching skill and invoke it immediately. If two skills could match, state both with one-line rationale and ask which to use. That is the only case where you may ask instead of act.

---

## Skill Map

### Product Pipeline

| Skill | When to invoke |
|---|---|
| `/preflight` | Validate environment before starting any non-trivial task |
| `/problem-discovery` | Structured problem framing before any spec or code — mandatory when solution space is open |
| `/prompt-builder` | Sharpen a feature brief before PRD or spec work begins |
| `/spec-creator` | Create a technical spec from a brief |
| `/doc-coauthoring` | Co-author structured documents — PRD, RFC, design doc |

### Engineering Pipeline

| Skill | When to invoke |
|---|---|
| `/tech-development-plan` | Sequenced implementation plan from user stories |
| `/10x-ai-engineer` | Full end-to-end feature pipeline: PRD → stories → plan → TDD → review → sync |
| `/secure-coding-practices` | Security posture review — always run before auth, API key, or Supabase changes |
| `/red-green-tdd` | Strict TDD execution (RED → GREEN → REFACTOR cycle) |
| `/evaluate-approach` | Compare competing implementation approaches before committing |
| `/pr-review` | Code review against PRD, stories, and acceptance criteria |
| `/context-sync` | Close a dev cycle; update CLAUDE.md with learnings |
| `/meta-learn` | Meta-learning synthesis across sessions or domains |

### Growth / GTM Pipeline

| Skill | When to invoke |
|---|---|
| `/growth-discovery` | Map the marketing lay of the land — martech, budget, personas, scale, channel — before any brief. Mandatory when audience, channel, or objective is open |
| `/marketing-brief` | Turn a strategy map into a rock-solid, analytics-grounded brief with measurable KPIs — the campaign contract |
| `/campaign-insights` | Improve a brief with data — ingest campaign / WhatsApp / web analytics, return a prioritised Brief Delta |
| `/marketing-automation-builder` | Turn an approved brief into an executable journey rendered as a beautified HTML workflow with KPIs wired in |

### Skill Stack Management

| Skill | When to invoke |
|---|---|
| `/skill-creator` | Create, edit, or improve any SKILL.md in the stack |
| `/autoresearch` | Benchmark and optimize a skill via eval loops |
| `/astack` | Route to the right skill; or `list` to see this map |

### Research & Signals

| Skill | When to invoke |
|---|---|
| `/data-analysis` | Data analysis or visualisation from any structured dataset |
| `/advanced-search` | Deep codebase or document search spanning multiple locations |

### Document & Artifact Generation

| Skill | When to invoke |
|---|---|
| `/pdf` | PDF reading, extraction, or analysis |
| `/xlsx` | Spreadsheet input or analysis |
| `/pptx` | PowerPoint file tasks |
| `/docx` | Word document tasks |
| `/web-artifacts-builder` | Multi-component claude.ai artifacts (HTML/CSS/JS) |
| `/doc-coauthoring` | Co-author structured documents — PRD, RFC, design doc |
| `/mcp-builder` | Create or improve an MCP server |
| `/brand-guidelines` | Apply Anthropic brand colours and typography |

---

## Routing Rules

1. **Engineering pipeline routing.** Full end-to-end feature → `/10x-ai-engineer`. Single-file / known root cause → direct implementation.

2. **Growth pipeline routing.** Strategy / audience / channel still open → `/growth-discovery` first. Have a strategy, need the brief → `/marketing-brief`. Have past performance data → `/campaign-insights`. Have an approved brief, need the journey → `/marketing-automation-builder`. The arc mirrors the build pipeline: discover → contract → data → artifact.

3. **Research routing.** Structured data → `/data-analysis`. Codebase search → `/advanced-search`.

4. **When ambiguous.** State the two best matches with one-line rationale each. Ask one question to resolve. Never guess silently.

5. **Stack management.** Creating or improving any skill → `/skill-creator` first, then mirror to both path locations (see CLAUDE.md skill deployment rule).
