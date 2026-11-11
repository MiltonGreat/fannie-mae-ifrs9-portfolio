# Code, Analysis, and Results — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-CAR-001  
**Version / Status:** 0.1 / Draft  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Code, Analysis, and Results (CAR)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

## 1. Purpose and scope

This document is the index to the Fannie Mae IFRS 9 ECL pipeline: the notebooks, the artifacts each produces, the run IDs, and the content hashes. It is the document a reader opens when they want to **verify that the results reported in the framework document (MAP-STF-FM-001) were produced by the pipeline** — not by hand, not by a separate computation, and not by an earlier version of the notebooks.

**This is not a validation opinion.** The Fannie Mae project is a framework, not a validated model. This document records what the pipeline computes and where the evidence lives. It does not interpret the results — the framework document does that.

**This is not a data dictionary.** The field-level dictionary is in `powerbi_exports/data_dictionary.md`. This document indexes the notebooks and their outputs, not the individual columns.

---

## 2. Notebook index

The pipeline has six notebooks, run in order. Each notebook asserts the presence of its upstream contract at start; a missing upstream file halts the notebook with a `FileNotFoundError` rather than silently falling back to defaults.

| # | Notebook | Purpose | Key outputs | Run ID (this run) |
|---|---|---|---|---|
| 01 | Data Loading | Quality & cleaning | Cleaned dataset (100,000 × 175); missing indicators; assumptions register; `data_quality_summary.json` | `20261021_121434` |
| 02 | EDA | Risk factor analysis | Risk drivers; correlations; vintage patterns; t-test results | `20261021_121539` |
| 03 | Calibration | PD/LGD/EAD calibration | IFRS 9 parameters; stage classification; LGD/EAD construction; ECL per stage and per scenario; `notebook_03_parameters.json` | `20261021_121550` |
| 04 | Stress Testing | Scenarios & coverage | Scenario ECLs; PD-cap sensitivity; severity check; coverage diagnostic; fairness assessment (UNRELIABLE at high cardinality) | `20261021_121552` |
| 05 | Reporting | Dashboard & report | Compliance report (with `[ANALYST: ...]` placeholders); decision log; manifest | `20261021_121553` |
| 06 | Power BI Export | Data export | Fact and dimension tables; data dictionary; manifest | `20261021_121555` |

**On the run IDs.** The six run IDs above are the ones the FDL, DCR, and CFR were frozen at (per their document-control tables). A re-run with the same seed and the same inputs produces the same content hash; the run IDs change because they encode the timestamp.

---

## 3. Artifacts by notebook

### 3.1 — NB01 (Data Loading)

| Artifact | Path | Purpose |
|---|---|---|
| Cleaned dataset | `data/fannie_mae_final_clean.csv` | The 100,000 × 175 clean frame; content hash `bc5263ca06d29037` |
| Assumptions register | `data/assumptions_register.csv` | The 10 assumptions the pipeline rests on |
| Missing-indicator columns | (within the clean frame) | `*_missing` flags created before imputation |
| Data quality summary | `outputs/reports/data_quality_summary.json` | The machine contract for NB04 and NB05 |
| Feature inventory | `data/feature_inventory.csv` | The include/exclude decisions for NB02 |
| Temporal audit | `outputs/reports/temporal_audit.csv` | Feature-by-feature classification by availability at decision time |
| Leakage findings | `outputs/reports/leakage_findings.csv` | The 9 confirmed leakage features |
| Findings register (NB01) | `outputs/reports/findings_register.csv` | NB01's 3 data-preparation findings |
| Data quality report | `outputs/reports/data_quality_report.txt` | The human-readable summary |

### 3.2 — NB02 (EDA)

| Artifact | Path | Purpose |
|---|---|---|
| Univariate statistics | `outputs/nb02_univariate_stats.csv` | Descriptive statistics for the key risk drivers |
| Outlier summary | `outputs/nb02_outlier_summary.csv` | 3× IQR outlier detection per feature |
| Segment analysis | `outputs/nb02_segment_analysis.csv` | Default rates by FICO band, LTV band, property type, occupancy, age band |
| Correlation comparison | `outputs/nb02_correlation_comparison.csv` | Pearson vs. Spearman correlations with `default` |
| Vintage analysis | `outputs/nb02_vintage_yearly.csv` | Default rates by origination vintage |
| T-test results | `outputs/nb02_ttest_results.csv` | Welch's t-test for each key risk driver |
| Visualizations | `outputs/figures/eda/*.png` | Univariate dashboard, correlation heatmap, vintage dashboard, boxplot comparison |
| Documentation | `outputs/*_analysis_documentation.txt` | Narrative documentation per analysis section |

### 3.3 — NB03 (Calibration)

| Artifact | Path | Purpose |
|---|---|---|
| Parameters JSON | `outputs/reports/notebook_03_parameters.json` | The machine contract for NB04 and NB05: PD, LGD, EAD, stage distribution |
| Limitations CSV | `outputs/reports/notebook_03_limitations.csv` | L1–L10; the methodology's documented limitations |
| Calibration summary | `outputs/reports/pd_lgd_ead_calibration_summary_v1_2.txt` | The human-readable PD/LGD/EAD/ECL summary |

### 3.4 — NB04 (Stress Testing)

| Artifact | Path | Purpose |
|---|---|---|
| Results JSON | `outputs/reports/notebook_04_results.json` | The machine contract for NB05: scenario ECLs, PD-cap sensitivity, severity check, coverage metrics, fairness results |
| Limitations CSV | `outputs/reports/notebook_04_limitations.csv` | L1–L6; the stress-testing limitations |
| Fairness limitations CSV | `outputs/reports/notebook_04_fairness_limitations.csv` | F-L1–F-L5; the fairness-specific limitations |

### 3.5 — NB05 (Reporting)

| Artifact | Path | Purpose |
|---|---|---|
| Compliance report | `outputs/reports/ifrs9_compliance_report_v1_2.txt` | The compliance report with `[ANALYST: ...]` placeholders |
| Assumptions narrative | `outputs/assumptions_register_narrative.md` | The human-readable version of the assumptions register |
| Decision log | `outputs/reports/decision_log_v1_2.txt` | The 25 documented decisions |
| Manifest | `outputs/reports/manifest_pipeline_v1_2.csv` | Every analysis artifact, with hash |

### 3.6 — NB06 (Power BI Export)

| Artifact | Path | Purpose |
|---|---|---|
| Fact tables | `powerbi_exports/fact_*.csv` | The six fact tables |
| Dimension table | `powerbi_exports/dim_segment.csv` | The segment dimension |
| Summary metrics | `powerbi_exports/summary_metrics.csv` | The dashboard header |
| Data dictionary | `powerbi_exports/data_dictionary.md` | Column-level documentation with limitations embedded |
| Manifest | `powerbi_exports/manifest.csv` | Every export, with hash |

---

## 4. Content hashes and reproducibility

### 4.1 — Key content hashes

| Artifact | Hash | Note |
|---|---|---|
| Clean frame (`fannie_mae_final_clean.csv`) | `bc5263ca06d29037` | The reproducibility check for NB01's output |
| Parameters JSON (`notebook_03_parameters.json`) | `[ANALYST: insert from manifest]` | Recorded in the manifest |
| Results JSON (`notebook_04_results.json`) | `[ANALYST: insert from manifest]` | Recorded in the manifest |
| Manifest (`manifest_pipeline_v1_2.csv`) | `[ANALYST: insert from audit log]` | The manifest is self-referential; its hash is recorded in NB05's audit log |

### 4.2 — Reproducibility

Every notebook run produces an audit log, a `run_id` on every artifact, and a SHA-256-16 hash on every saved file. The same random seed (42) is used throughout. **Same data + same seed → same artifacts.**

**What is not reproducible without re-running the pipeline:**
- Model pickles (none exist in the FM pipeline — the parameters are computed, not fitted)
- The raw source file (`Fannie Mae.zip` → `train4.csv`, 1,672,822 rows) — only the 100,000-row sample is retained

**What is reproducible:**
- The clean frame's content hash
- Every artifact's file hash
- Every run ID
- Every audit log

### 4.3 — The audit log

Each notebook produces its own audit log in `logs/`, with a distinct filename prefix:

| Notebook | Log filename |
|---|---|
| NB01 | `logs/eda_audit_trail_*.log` |
| NB02 | `logs/eda_audit_trail_*.log` **(see note)** |
| NB03 | `logs/calibration_audit_trail_*.log` |
| NB04 | `logs/stress_testing_audit_trail_*.log` |
| NB05 | `logs/reporting_audit_trail_*.log` |
| NB06 | `logs/powerbi_export_audit_trail_*.log` |

> **A note on the FM NB01 and NB02 log prefix collision.** Both NB01 and NB02 use `eda_audit_trail_*.log` as their prefix. A reviewer finding a log by filename alone cannot tell whether it belongs to NB01 or NB02. **This is a known issue**, and the FM README's Reproducibility Record (§4 of `deliverables/05_reproducibility_record.md`) documents it. The recommended fix is a MINOR version bump across the pipeline (prefix NB02's log with `model_dev_`).

---

## 5. Notebook dependency chain

The pipeline's dependency graph:

```
NB01 (Data Loading)
  ├── data/fannie_mae_final_clean.csv
  ├── data/feature_inventory.csv
  └── outputs/reports/data_quality_summary.json
        │
        ▼
NB02 (EDA) — reads the clean frame; no upstream JSON dependency
        │
        ▼
NB03 (Calibration)
  ├── reads the clean frame and feature inventory
  ├── writes outputs/reports/notebook_03_parameters.json
  └── writes outputs/reports/notebook_03_limitations.csv
        │
        ▼
NB04 (Stress Testing)
  ├── reads notebook_03_parameters.json (asserts authored_by)
  ├── writes outputs/reports/notebook_04_results.json
  ├── writes outputs/reports/notebook_04_limitations.csv
  └── writes outputs/reports/notebook_04_fairness_limitations.csv
        │
        ▼
NB05 (Reporting)
  ├── reads notebook_03_parameters.json and notebook_04_results.json
  ├── asserts NB04's upstream_nb03_run_id matches NB03's run_id
  ├── writes outputs/reports/ifrs9_compliance_report_v1_2.txt
  ├── writes outputs/reports/decision_log_v1_2.txt
  └── writes outputs/reports/manifest_pipeline_v1_2.csv
        │
        ▼
NB06 (Power BI Export)
  ├── reads notebook_03_parameters.json and notebook_04_results.json
  ├── asserts NB04's upstream_nb03_run_id matches NB03's run_id
  └── writes powerbi_exports/*.csv and manifest.csv
```

**Every notebook's pre-flight check is a `FileNotFoundError` if an upstream artifact is missing.** The pipeline does not fall back to synthetic data or default values.

---

## 6. Where the framework document's numbers come from

Every headline number in the framework document (MAP-STF-FM-001) traces to a specific notebook artifact. The cross-reference:

| Framework document number | Value | Source artifact | Source field |
|---|---|---|---|
| Default rate | 0.94–0.95% | `data_quality_summary.json` | `default_rate` |
| PD (unconditional) | 0.95% | `notebook_03_parameters.json` | `results.unconditional_pd` |
| Average LGD | 44.28% | `notebook_03_parameters.json` | `results.average_lgd` |
| Average EAD | $204,745 | `notebook_03_parameters.json` | `results.average_ead` |
| Stage 1 / 2 / 3 | 89.1% / 9.9% / 1.0% | `notebook_03_parameters.json` | `results.stage_distribution` |
| ECL per loan (weighted) | $2,231 | `notebook_04_results.json` | `total_weighted_ecl` |
| Portfolio ECL (weighted) | ~$223M | (derived: `$2,231 × 100,000`) | — |
| EAD ratio | 86.95% | `notebook_04_results.json` | `ecl_ratio_of_ead` (derived) |
| Coverage (weighted) | 22.31% | `notebook_04_results.json` | `capital_coverage.ecl_pct_of_assets_weighted` |
| Coverage (tail) | 75.12% | `notebook_04_results.json` | `capital_coverage.ecl_pct_of_assets_tail` |
| Severity ratio | 1.27× | `notebook_04_results.json` | `severity_check.ratio_vs_historical` |
| PD-cap effect | +0.0% | `notebook_04_results.json` | `pd_cap_sensitivity.cap_effect_pct` |

**On the traceability.** A reader who wants to verify any framework-document number follows the cross-reference to the artifact, opens the artifact's JSON, and finds the field. **Every number in the framework document is traceable to a specific field in a specific JSON artifact.** This is the pipeline's equivalent of the Lending Club pipeline's `authored_by: "NB03"` chain.

---

## 7. Known issues and open items

| Issue | Where it appears | Status |
|---|---|---|
| NB01/NB02 audit log prefix collision | NB01 and NB02 both use `eda_audit_trail_*.log` | Documented; fix is a MINOR version bump |
| Reconciliation warning on `base_pd` (definitional, not a defect) | NB05 (and NB04 earlier) log a warning when `df['pd_estimate'].mean() = 5.8%` differs from `unconditional_pd = 0.95%` | Documented in the README's Claim-Authorization section; should be reclassified as INFO and added to the Findings Register |
| The `_v1_2` filename for the PD/LGD/EAD calibration summary | NB03 writes `pd_lgd_ead_calibration_summary_v1_2.txt`; the README's "Key documents" section now matches | Fixed in the README revision |

**The reconciliation warning is the most interesting item.** The 5.8% vs. 0.95% difference is **definitional** — `pd_estimate` is a per-loan score with a range of [0.01, 0.10], and its mean over the portfolio is 5.8%; `unconditional_pd` is the portfolio default rate (0.95%). The two numbers measure different things. The warning is correct to fire (it says "the dataframe's mean and the contract's value differ"), but the "expected" reading should be documented so a future reviewer does not investigate a definitional difference as a data defect. **This is documented in the README, not in the notebook's audit log.**

---

## 8. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-CAR-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | At the next version bump of any upstream notebook |

---

## 9. What this document contains, and what it does not

### What it contains (from the notebook output)

- The six-notebook index with run IDs.
- The artifacts each notebook produces.
- The content hashes (one filled, two marked).
- The audit-log filenames and the prefix-collision note.
- The dependency graph.
- The cross-reference from every framework-document number to its source artifact.
- The known issues and open items.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§4.1** — the parameters/results/manifest hashes, which are recorded in the manifest and audit log.
- **§8** — the document owner's name and role.

**None of these are gaps in the notebook output.** They are values that exist in the repository's artifacts but are not reproduced in this document. A reader fills them by opening the referenced artifact.

**When the markers are filled, the document is complete.** It is the most notebook-sourced of the four documents in this batch and is publishable as-is once the three hash markers and one name field are filled.
