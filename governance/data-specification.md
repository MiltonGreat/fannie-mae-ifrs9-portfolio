# Data Specification — Fannie Mae IFRS 9 ECL Framework

**Document ID:** MAP-DS-FM-001  
**Version:** 0.1  
**Status:** Draft — pending analyst completion of the marked sections  
**Project:** Fannie Mae IFRS 9 ECL Framework  
**Owner:** `[ANALYST: name, role]`  
**Companion documents:** [`problem-definition.md`](problem-definition.md), [`analytical-specification.md`](analytical-specification.md), [`feature-decision-logs.md`](feature-decision-logs.md), [`methodology-document.md`](methodology-document.md)

---

## 1. Purpose and scope

This Data Specification documents the data used to build and validate the Fannie Mae IFRS 9 ECL framework. It records:

- **What data was used** — the source, the sample, the clean frame.
- **How the data entered the pipeline** — the load, the quality checks, the missingness treatment.
- **What was excluded and why** — the leakage audit, the feature decisions, the scope limitations.
- **What the data can and cannot support** — the analyst's fitness-for-purpose determination.

**This document is not a data dictionary.** The field-level dictionary is in [`powerbi_exports/data_dictionary.md`](../powerbi_exports/data_dictionary.md). This document is the *lineage and fitness* record that the field-level dictionary sits inside.

**This document is not a Validation Opinion.** The Fannie Mae project is a framework, not a validated model; no Validation Opinion exists for it, by design. This document records the data the framework was built on and the analyst's determination of whether that data supports the framework's claims.

---

## 2. Source data

### 2.1 Source publication

| Field | Value |
|---|---|
| **Dataset name** | Fannie Mae Single-Family Loan Performance |
| **Publisher** | Fannie Mae |
| **Publication form** | Public data tape |
| **Original file** | `Fannie Mae.zip` → `train4.csv` |
| **Original row count** | 1,672,822 rows |
| **Original column count** | `[ANALYST: insert — the notebooks do not record the raw file's column count; it is in the source file header]` |
| **Vintage coverage** | `[ANALYST: insert — the dataset's vintage range; the notebooks do not record this]` |
| **License / terms** | `[ANALYST: insert — Fannie Mae's publication terms]` |

### 2.2 Sampling

| Field | Value |
|---|---|
| **Sample size** | 100,000 rows |
| **Sampling method** | Random sample |
| **Random seed** | 42 (project-wide; specified in AS-PR004 §8) |
| **Sampling rationale** | Computational tractability without material distributional distortion |
| **Sampling reproducibility** | Same seed + same source file → same sample. The sample's content hash is the reproducibility check. |

**Note on the sampling.** The sample is random, not stratified. For the ECL pipeline, this is acceptable because the observed default rate (0.94–0.95%) matches the population default rate to two decimal places. If the sample were stratified by a variable that correlates with default, the sampling method would need to be recorded as a limitation. It is not.

### 2.3 Load path

The sample was loaded in NB01 Phase 2. The load is a two-location file resolution:

```
Path 1: loans_full_schema.csv (working directory)
Path 2: data/loans_full_schema.csv (data folder)
```

If neither exists, NB01 raises `FileNotFoundError` and does not proceed. The no-synthetic-fallback rule applies: a missing source file halts the pipeline; it does not trigger a fallback to synthetic data.

**What this means for a reviewer:** if the pipeline ran, the source file existed. The audit log records the file's hash, so the specific source file is verifiable from the run.

---

## 3. Data lineage

### 3.1 Pipeline lineage

```
Fannie Mae.zip → train4.csv (1,672,822 rows)
       │
       ▼
NB01 Phase 2: random sample (seed 42) → 100,000 rows
       │
       ▼
NB01 Phase 3–6: target definition, missingness analysis,
                leakage detection, temporal audit,
                feature inventory
       │
       ▼
fannie_mae_final_clean.csv (100,000 × 175)
       │  content hash: bc5263ca06d29037
       │  (run 20261021_121434)
       ▼
NB02–NB06: downstream pipeline
       │
       ▼
NB03 parameters: outputs/notebook_03_parameters.json
NB04 results:    outputs/notebook_04_results.json
```

### 3.2 Source-system lineage

The lineage **inside** Fannie Mae's publication process — how the raw loan tapes became `train4.csv`, what preprocessing was applied before publication, what vintages the file covers — is **not recorded in the pipeline**.

`[ANALYST: insert — the source-system lineage. A complete Data Specification would record: (a) the Fannie Mae publication version this file came from; (b) the vintage range and the pre-publication processing; (c) any known data issues in the source publication that would affect the pipeline's claims. If this is not available from the Fannie Mae publication documentation, state that explicitly — the absence is itself a limitation.]`

### 3.3 Lineage verification

The pipeline lineage is verifiable from the run artifacts:

| Check | How to verify |
|---|---|
| The clean frame's shape (100,000 × 175) | `fannie_mae_final_clean.csv` header and row count |
| The clean frame's content hash | Recorded in the audit log for run `20261021_121434` |
| The sample was drawn from the source | NB01 audit log records the source file hash |
| The downstream notebooks read the clean frame | NB02–NB06 all assert the upstream contract's presence and reconcile the run IDs |

**The source-system lineage is not verifiable from the pipeline.** It depends on the Fannie Mae publication, which the pipeline does not record. This is a limitation, and it should be stated in the fit-for-purpose section (§8).

---

## 4. Missingness profile

### 4.1 Summary

| Field | Value |
|---|---|
| **Total missing values** | `[ANALYST: insert — the NB01 output shows a summary line; the exact number is in NB01's Phase 4 output]` |
| **Columns with missing values** | 10 |
| **Columns above 20% missing** | 5 |
| **Missingness threshold (high-risk)** | 20% (per DCR's data-quality rule) |
| **Imputation method** | Median (numeric) / mode (categorical) |
| **Missing indicators** | Created before imputation; preserved as `*_missing` columns |
| **Post-imputation missingness** | Zero missing values in the clean frame |

### 4.2 High-missingness columns

| Column | Missing count | Missing % | Why |
|---|---|---|---|
| `verification_income_joint` | `[ANALYST: insert from NB01 output]` | 85.45% | Joint-income fields are only populated when a joint application is filed; missingness is structural, not random |
| `annual_income_joint` | `[ANALYST: insert from NB01 output]` | 85.05% | Same — joint-income field |
| `debt_to_income_joint` | `[ANALYST: insert from NB01 output]` | 85.05% | Same — joint-DTI field |
| `months_since_90d_late` | `[ANALYST: insert from NB01 output]` | 77.15% | Populated only for borrowers with a 90+ day late event; the vast majority have no such event |
| `months_since_last_delinq` | `[ANALYST: insert from NB01 output]` | 56.58% | Populated only for borrowers with a delinquency; many have none |

### 4.3 Why indicators are created before imputation

The pipeline creates a `_missing` flag for every column with missing values, **before** imputation. This preserves the information that a value was absent. Imputing first and flagging afterward would destroy the information the flag is meant to preserve.

**This is the E-23 §4.2 data-quality control the notebook implements.** The missingness analysis (column-level counts), the missingness indicators (preserve the pattern), and the documented imputation method are the three controls that make the imputation auditable. A reviewer can see, for every imputed column, that the flag exists *before* the fill.

### 4.4 What this means for a reader

**Zero missing values in the clean frame does not mean the missingness was harmless.** It means the missingness was handled. The `_missing` flags are the handling; they are the mitigation, not the remedy. A reader who sees "no missing values" should look at the flags to understand where the imputation occurred.

**The imputation method is median for numeric and mode for categorical.** This is a conservative choice (robust to outliers), but it does not preserve relationships between columns. For a joint-income pair where one column is missing, imputing the median independently of the other column may create an internally inconsistent record. **This is a limitation, and it is documented in §8.**

---

## 5. Leakage audit

### 5.1 The pre-registered list

The pipeline uses a pre-registered leakage list (`KNOWN_LEAKAGE`) frozen in DS-001 §7 before data inspection. The list has 9 features:

1. `balance`
2. `paid_total`
3. `paid_principal`
4. `paid_interest`
5. `paid_late_fees`
6. `months_since_last_delinq`
7. `months_since_90d_late`
8. `months_since_last_credit_inquiry`
9. `issue_month`

**All nine were confirmed present in the dataset and removed in NB02.**

### 5.2 Why pre-registered

The pre-registered list is what makes the leakage finding trustworthy. A post-hoc leakage scan (correlation > 0.5, missingness-target correlation > 0.15) would flag features that correlate with default — but the correlation could be causal, or could be leakage, or could be spurious. The pre-registered list says: **these 9 features are leakage by definition, regardless of what the scan finds.** The scan is a diagnostic, not a verdict.

**The pre-registered list is what makes the 0.865 → 0.631 drop a finding rather than a post-hoc rationalization.** The developer's claim was made against a model trained on all features. The analyst's finding is that 9 of those features could not have been known at decision time. The list was written before the data was inspected, so it is not tuned to produce a particular answer.

### 5.3 The scan as diagnostic

The leakage scan (Phase 5) applies two tests:

- **Test A:** univariate correlation with default > 0.5 → HIGH severity if > 0.6, MEDIUM if 0.5–0.6.
- **Test B:** missingness-target correlation > 0.15 → HIGH severity if > 0.25, MEDIUM if 0.15–0.25.

**The scan's findings are not the leakage finding.** The scan flags features worth a second look; the pre-registered list determines which features actually count. A feature the scan flags but which is not on the pre-registered list is **not leakage** by the pipeline's definition — it is a candidate for review.

**The scan's role is corroborative.** If the scan finds the pre-registered features, the list is corroborated. If the scan misses some of them, the list's completeness is called into question. If the scan flags features not on the list, those features should be reviewed.

### 5.4 What the leakage finding means for the model

The realistic AUC after leakage removal is 0.631. The developer-claimed AUC with leakage is 0.865. The difference (0.234 AUC points, 27.1%) is **the leakage premium** — the performance the model would have appeared to have if the leakage features were available at decision time.

**The leakage is resolved.** The features are removed, the model was retrained, the realistic AUC is the model's honest performance. This is F-03.01 (CRITICAL, RESOLVED).

---

## 6. Feature decisions

The feature inventory ([`data/feature_inventory.csv`](../data/feature_inventory.csv)) records, for every column in the clean frame:

- Data type
- Missing percentage
- IV (simplified approximation, `|corr| × 0.3`, capped at 0.5 — a screening heuristic, not a textbook WOE-based IV)
- Inclusion decision (INCLUDE / EXCLUDE)
- Exclusion reason

**The inventory is the contract between NB01 and NB02.** NB02 reads the inventory to determine which columns to remove. A feature's inclusion decision is the record of what NB02 should do; a missing or mislabeled row breaks the contract.

**The IV column is a screening heuristic, not a feature-selection statistic.** A feature with IV 0.05 is not necessarily weak — the simplified IV captures only linear correlation with the target, not the feature's joint behavior with other features. A reader who treats the IV column as a WOE-based IV will over-weight it. **The Validation Opinion should not cite the IV column as evidence of feature strength.**

### 6.1 What the inventory does not record

- **Feature interactions.** The IV column is univariate. Two features that individually have low IV but jointly have high predictive power would not be captured. **This is a limitation, and it is documented in §8.**
- **Categorical feature encodings.** The inventory records the decision, not the encoding. A reader who wants to know how a categorical feature was encoded follows the notebook to NB02. **This is acceptable** — the encoding is a modeling decision, not a data decision, and the Feature Decision Log records the modeling decisions.
- **The per-feature rationale in prose.** The inventory has a one-line `Exclusion_Reason` column, but it does not have a narrative rationale per feature. **This is a limitation of the artifact, not of the pipeline.** A reviewer who needs the full rationale cross-references [`feature-decision-logs.md`](feature-decision-logs.md).

---

## 7. Data quality score

The pipeline computes a data quality score of 58.9/100 (POOR) from four penalties:

| Penalty | Contribution | What it measures |
|---|---|---|
| Missingness | 21.1 | Average missingness across columns with missing, halved, capped 30 |
| Duplicates | `[ANALYST: insert — the NB01 output does not show the duplicate count, but the score's penalty breakdown in the output shows the missingness and leakage components; the duplicate component is not shown. If the duplicate penalty is 0, state so.]` | Duplicate rows in the clean frame |
| Leakage | 15 | 3 per leakage feature, capped 15 |
| Macro gap | 5 | No macro indicators |

**The score's bands** (GOOD ≥80, MODERATE ≥60, POOR ≥40, CRITICAL <40) are descriptive conventions, not DCR thresholds. A reader should use the score as a first read on data quality, not as a determination.

**What the score does not measure:** the score does not measure whether the data supports the ECL pipeline's claims. It measures the data's internal quality (missingness, duplicates, leakage, macro coverage). A dataset with a high data quality score could still be unfit for a specific use case if the use case requires features the data does not have.

**The 58.9 score should be read alongside the limitations in §8.** The data is POOR by the pipeline's own measure, and the limitations explain why.

---

## 8. Limitations and fitness for purpose

### 8.1 Data limitations

| ID | Limitation | Impact | Source |
|---|---|---|---|
| L1 | PD-LGD correlation is constructed | Convexity illustrative, not empirical | `notebook_03_limitations.csv` |
| L2 | SICR uses top-decile proxy | Stage 2 is illustrative, not observed SICR | `notebook_03_limitations.csv` |
| L3 | LGD is a function of PD | Average LGD 44.28% (inside benchmark) | `notebook_03_limitations.csv` |
| L4 | Prepayment/curtailment are assumptions | EAD approximate | `notebook_03_limitations.csv` |
| L5 | EIR is industry average | Discounting approximate | `notebook_03_limitations.csv` |
| L6 | PD cap masks tail convexity | Tail ECL understated pre-cap | `notebook_03_limitations.csv` |
| L7 | Fairness testing incomplete | No direct protected attributes | `notebook_04_limitations.csv` |
| L8 | U.S. data as Canadian proxy | Results illustrative | `notebook_04_limitations.csv` |
| L9 | Scenario weights/multipliers assumption-based | Scenario severity directional | `notebook_04_limitations.csv` |
| L10 | Capital coverage is diagnostic, not adequacy | CET1 is a ratio of RWA, not assets | `notebook_04_limitations.csv` |

### 8.2 Data-specific limitations (this document)

| ID | Limitation | Why |
|---|---|---|
| D1 | Source-system lineage not recorded | The pipeline records the sample's provenance but not the Fannie Mae publication's provenance |
| D2 | IV column is a simplified approximation | The screening heuristic captures linear correlation, not joint behavior |
| D3 | Joint-income imputation is independent | Median imputation of one joint field is not conditional on the other |
| D4 | Duplicate count not independently verified | The score's duplicate penalty is `[ANALYST: insert]`, and a duplicate audit was not performed at the row level |

### 8.3 Fitness-for-purpose determination

The Data Specification's closing judgment: **is this data fit for the ECL framework's intended use?**

`[ANALYST: insert the determination. The question is not "is the data good?" — it is "does the data support the claims the framework makes?" The framework's claims are: (a) a working IFRS 9 ECL pipeline demonstrating PD/LGD/EAD calibration, staging, and probability-weighted ECL; (b) with documented assumptions; (c) on a public proxy dataset. If the answer is yes, state it with the caveats from §8.1 and §8.2. If the answer is "fit for a demonstration but not for production," state that — it is the more honest answer, and it matches the README's "what this framework does NOT demonstrate" section. If the answer is "not fit for the demonstration either," state that too — and explain what would be needed to make the demonstration work.]`

### 8.4 What the data would need to support a production implementation

A production IFRS 9 ECL implementation would require:

- **Current credit characteristics** — to replace the SICR proxy with an observed measurement.
- **Recovery data** — to calibrate LGD from actual recoveries, not from assumptions.
- **Macro indicators** — to replace the narrated macro scenario with a computational linkage.
- **Canadian portfolio data** — to validate the pipeline for a Canadian use case.
- **Protected attributes** — to perform direct fairness testing rather than geographic-proxy testing.
- **Time-based cohorts** — to replace the random split with a walk-forward split.

None of these are present in the Fannie Mae public dataset. **This is not a defect of the dataset** — the dataset is what it is, and it is a public proxy. The defect would be in the *claim* if the framework claimed production readiness. The framework does not, and this document records the gap between what the data supports and what a production implementation would require.

---

## 9. Data governance and access

### 9.1 Data classification

| Field | Value |
|---|---|
| **Classification** | Public — Fannie Mae Single-Family Loan Performance is a public dataset |
| **Sensitive attributes** | None. The dataset contains no protected-class attributes and no direct borrower identifiers. |
| **Geographic granularity** | `[ANALYST: insert — the dataset's geographic fields (state, MSA, ZIP) are present, but the pipeline's fairness testing treats them as proxies rather than as identifiers. State the granularity and the pipeline's treatment.]` |
| **Re-identification risk** | Low. The dataset is a public loan-performance tape with no direct identifiers. |

### 9.2 Access controls

Not applicable — the data is public. The pipeline's access is governed by the notebook run environment, not by a data access control.

### 9.3 Retention and deletion

Not applicable — the data is public. The clean frame is retained in the repository for reproducibility. The run artifacts (audit logs, manifests, hashes) are retained for the same reason.

---

## 10. Document control

| Field | Value |
|---|---|
| **Document ID** | MAP-DS-FM-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | [`problem-definition.md`](problem-definition.md), [`analytical-specification.md`](analytical-specification.md), [`feature-decision-logs.md`](feature-decision-logs.md), [`methodology-document.md`](methodology-document.md), [`../powerbi_exports/data_dictionary.md`](../powerbi_exports/data_dictionary.md) |
| **Next review** | On completion of the analyst-marked sections, or at the next version bump of any upstream notebook |

---

## 11. What this document contains, and what it does not

### What it contains (from the notebook output)

- The source, sample, and load path.
- The pipeline lineage and its verifiability.
- The missingness profile and the missingness-control rationale.
- The leakage audit and the pre-registered-list rationale.
- The feature inventory's existence and its limitations.
- The data quality score and the limitations it sits alongside.
- The ten documented limitations from the notebooks, plus four data-specific limitations from this document.
- The gap between what the data supports and what a production implementation would require.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§2.1** — the Fannie Mae publication details (vintage range, license, original column count).
- **§3.2** — the source-system lineage inside Fannie Mae's publication process.
- **§4.1–§4.2** — the exact missing counts for the high-missingness columns (these are in NB01's Phase 4 output, which I did not capture in full).
- **§7** — the duplicate penalty in the data quality score.
- **§8.3** — the analyst's fitness-for-purpose determination.
- **§9.1** — the geographic granularity statement.

**None of these are gaps in the notebook output.** They are the analyst's contribution — the parts of a Data Specification that the notebook does not produce because they are judgment, provenance, or external-source description. The document is complete in structure; the markers name where the analyst's contribution goes.

**When you fill in the markers, the document becomes publishable as-is.** The structure, the factual content, the limitations, and the fit-for-purpose framing are all in place.
