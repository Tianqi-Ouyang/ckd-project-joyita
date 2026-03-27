# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
### Data safty

Data Privacy & Compliance Protocol (HIPAA/GDPR)
This project involves sensitive hospital research data. Compliance with HIPAA (US) and GDPR (EU) is mandatory. All interactions with Claude must adhere to the following safety guardrails.

1. The "Zero-PHI" Principle
Never upload, paste, or pipe files containing Protected Health Information (PHI) or Personally Identifiable Information (PII) into the AI.

Prohibited Data: Names, social security numbers, exact dates (birth/discharge), medical record numbers, or specific geographic identifiers.

Allowed Data: synthetic data, or structural metadata (column names and types only). Ask user to confirm

2. Safe Coding Workflow
To maintain data integrity while using AI for statistical analysis:

Structural Prompting: Provide Claude with the structure of your Excel or text files (e.g., str(df) or the first 2 empty rows) rather than the data itself.

Synthetic Development: Request Claude users to run on themselves.

Local Execution: Once Claude provides the logic or script, run it on your local machine or secure hospital server where the real data resides. Do not return the output of the real data analysis to the AI if it contains individual-level records.

3. Handling Files (.xlsx, .csv, .txt)
Excel Files: can only read first line variable names, and ask user to confirm.

Text Files: ask for variable names.

4. Claude Configuration Requirements
Ensure the following settings are active in your Claude environment:

Data Training: Verify that "Product Improvement" (training on your chats) is Disabled in the account settings.

Session Context: If Claude starts "hallucinating" or requesting more data than necessary to solve a coding problem, reset the session to clear the context.




Do not read or upload any file to server. Do not access any personal inforamtion, like EMPI.
Do not change .Rmd files 
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
