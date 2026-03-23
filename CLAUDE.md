# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
Do not upload any EMPI, identified information, if needed just use fake value for id

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
- Exclusions: missing baseline CRE, prior ESRD/KT (main cohort only), eGFR < 10.
