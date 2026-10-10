---
name: data-analysis
description: Explore, clean, analyse, and visualise any structured dataset (CSV, TSV, JSON, Parquet, XLSX, SQL result). Use whenever the user shares a data file, asks "what does this data show", requests summary stats, wants a chart, needs cohort or segment analysis, asks to spot outliers, wants a trend or correlation, or needs a data-led answer before a decision. Also trigger for A/B test read-outs, campaign performance, funnel drops, cohort retention, revenue breakdowns, and any question that maps to a groupby-aggregate-plot workflow. Do not use for schema design, ETL pipeline building, or ML model training — use `/contract-design`, `/telemetry-design`, or a dedicated ML skill instead.
---

# Data Analysis

Turn a raw dataset into a decision-grade answer. Never eyeball; always compute. Never dump raw tables; always summarise then chart.

The four-step loop: **profile → clean → analyse → present.** Skip a step only when the user names the exact slice they want.

---

## Behaviour rules

1. **Profile before analysing.** Do not aggregate a column you have not first inspected for nulls, type, range, and outliers.
2. **Compute; do not eyeball.** Every claim maps to a script or a query. If a claim is not backed by output, drop it.
3. **One chart per question.** Do not stack unrelated series.
4. **Cite the numerator and denominator.** Percentages without a base are misleading.
5. **State the confidence.** Every answer ends with a 0-100% score and what would raise it.

---

## Intake (ask once, batched)

Ask only what the file, extension, or user prompt does not already state:

1. What is the file path or paste?
2. What is the business question in one sentence?
3. What is the grain of one row? (one order, one user, one event, one day)
4. What is the time range and time-zone convention?
5. Which columns are keys, which are metrics, which are dimensions?
6. Are there known data-quality issues to expect? (nulls, duplicates, encoding)
7. What decision does the answer feed?

---

## Step 1 — Profile

Load with the right tool: `pandas.read_csv` for CSV/TSV, `pandas.read_excel` for XLSX, `pyarrow` for Parquet, `sqlite3` or `duckdb` for SQL. Then print:

- `df.shape`
- `df.dtypes`
- `df.head(3)`
- `df.isna().sum()`
- `df.describe(include="all")`
- Cardinality of each dimension column: `df[col].nunique()`
- Top 5 and bottom 5 values of every date/timestamp column

Save the profile to `analysis/00-profile.md`. Do not proceed to Step 2 until every column is understood.

---

## Step 2 — Clean

Apply only the transforms the profile justifies. Log every transform to `analysis/01-clean.md`.

Common transforms:
- Cast types (`pd.to_datetime`, `pd.to_numeric` with `errors="coerce"`)
- Deduplicate on key columns
- Impute or drop nulls — state which and why
- Bucket high-cardinality dimensions
- Normalise text (lower, strip, canonical spellings)
- Timezone-align timestamps

Never silently drop rows. Print `before → after` row counts after every drop.

---

## Step 3 — Analyse

Match the question type to the pattern:

| Question | Pattern |
|---|---|
| "What is the trend of X?" | Time series: resample daily/weekly/monthly, plot + 7-day MA |
| "What drives X?" | Groupby dimension, sort by X, show top and bottom |
| "How do A and B differ?" | Cohort compare, side-by-side aggregate, delta column |
| "Is X changing?" | Period-over-period: last N vs prior N, absolute + percent delta |
| "What is the distribution?" | Histogram or ECDF, report mean/median/p90/p99 |
| "Are A and B related?" | Scatter + Pearson/Spearman; state effect size, not just p-value |
| "Which segment is best?" | Rank table with base sizes; suppress segments below a minimum N |
| "Where does the funnel leak?" | Sequential rates: N at step k / N at step k-1; plot the survival curve |

Write every query as a script under `analysis/queries/`. Never run analysis inside a REPL only — write it once, run it many times.

---

## Step 4 — Present

One-page summary at `analysis/REPORT.md`:

- **Headline finding** — one sentence, with numbers.
- **What the data says** — 3-5 bullets, each with a number and a base.
- **Chart** — one primary chart, saved to `analysis/charts/headline.png`.
- **Caveats** — data-quality issues that could change the answer.
- **What we would do next** — one recommended action.
- **Confidence: X%** — and what would raise it.

For richer output, produce a self-contained HTML artifact using `/web-artifacts-builder` or the `dataviz` skill for chart discipline.

---

## Anti-patterns to reject

- Averaging without weighting when row grain is uneven.
- Reporting a segment's rate without its N.
- Comparing periods of different lengths.
- Using percent-change on a base that includes zero.
- Presenting three charts when one would answer the question.

---

## Output structure

```
analysis/
├── 00-profile.md
├── 01-clean.md
├── queries/
│   ├── trend.py
│   ├── segment.py
│   └── ...
├── charts/
│   └── headline.png
└── REPORT.md
```

---

## Confidence score

Close every report with:
`Analysis confidence: X% — [N] questions answered against [M] rows after cleaning. [Gaps if data was partial.]`
