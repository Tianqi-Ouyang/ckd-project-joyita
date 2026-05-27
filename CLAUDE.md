# CLAUDE.md

@~/.claude/hospital-privacy.md

## Project Overview

Clinical research analysis of **long-term kidney outcomes after ICI therapy** (CKD incidence, progression, ESKD) over 10-year follow-up. Author: Tianqi Ouyang.

## How to Render

```bash
quarto render analysis_ici_ckd.qmd --to html   # main cohort
quarto render treated_group.qmd --to html       # treated group descriptive
```

## Files

| File | Description |
|---|---|
| `analysis_ici_ckd.qmd` | Main cohort analysis (master Excel + RPDR Lab) |
| `treated_group.qmd` | Treated group descriptive (treated group.xlsx + RPDR Lab) |
| `treated group.xlsx` | Input: 393 ICI-treated patients (11 columns) |
| `treated_group.xlsx` | Output: enriched with derived variables |
| `check_master_2025.xlsx` | Supplementary data |

## Key Conventions

- All time anchors on `Index_date` (ICI start). `ICI_Dose_Date` renamed to `Index_date`.
- **Follow-up**: `last_cre_date` → `last_cre_days` for censoring.
- Yearly windows: year X = days `(X*365 - 90)` to `(X*365)` post-index.
- Survival flags: `survival_Xyear = 1` if `last_cre_days >= 365 * X`.
- Outcome flags: `{outcome}_Xyear = 1` if `time_to_{outcome} < 365 * X`, NA→0, factor.
- eGFR: CKD-EPI 2021 creatinine-only (race-free).
- `pre_lab_median_year` falls back to closest lab if no labs in 90-day window.
- CKD incidence: >25% eGFR drop AND eGFR <60, from baseline >=60, sustained >=90 days.
- CKD progression: >30% eGFR drop from baseline <60, sustained >=90 days.
- ESKD definition 1: eGFR <10, sustained >=90 days.
- Competing risk: `time_to_ckd_cmp = pmin(time_to_ckd_composite, last_cre_days, death_days)`; priority: death(2) > CKD(1) > censored(0).
- Exclusions: missing baseline CRE, prior ESRD/KT (main cohort only), eGFR < 10.

## Coding Style — ASCII only in gtsummary strings

`gtsummary` (and its underlying `gt` HTML renderer in Quarto) escapes some non-ASCII characters to literal `<U+xxxx>` codes inside rendered tables. **Always use ASCII** in any string that ends up inside a gtsummary call — especially `modify_caption()`, the `label = list(...)` argument to `tbl_regression()`/`tbl_summary()`, and the `type = list(...)` labels.

Use these substitutions everywhere inside gtsummary calls (and prefer them in all code/markdown for consistency):

| Avoid | Use instead |
|---|---|
| `—` (em dash) / `–` (en dash) | `-` |
| `×` (multiplication) | `x` |
| `²` (superscript 2) | `2` (e.g. `m2` not `m²`) |
| `≥` / `≤` | `>=` / `<=` |
| `±` | `+/-` |
| `μ` | `u` (e.g. `umol/L`) |

Plain markdown body text (headings, paragraphs, bullets) outside gtsummary calls can use Unicode — Quarto/HTML renders it correctly. The rule only matters for strings passed into gtsummary.

### Also avoid `space - space` inside ggplot `title` / `subtitle`

R's default PNG font renderer (Cairo / Quartz on macOS) interprets a freestanding hyphen (` - `) between words inside a plot title as a typographic em-dash (U+2014), and falls back to printing the literal `<U+2014>` glyph if the font lacks it. Use a colon or rephrase instead:

| Avoid | Use |
|---|---|
| `"CIF - CKD Composite by group"` | `"CIF: CKD Composite by group"` |
| `"Kaplan-Meier - All-cause mortality"` | `"Kaplan-Meier: All-cause mortality"` |
| `"Model-predicted eGFR - 3 years"` | `"Model-predicted eGFR over 3 years"` |

Hyphenated compounds with no surrounding spaces (e.g. `Model-predicted`, `Kaplan-Meier`, `long-term`) are fine — only the freestanding ` - ` separator is replaced.
