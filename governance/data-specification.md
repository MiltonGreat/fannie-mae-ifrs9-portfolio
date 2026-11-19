# Data Specification (A03)

| Metadata Field | Value |
| :--- | :--- |
| **Document ID** | `MAP-DS-FM-001` |
| **Version** | `0.2.0-draft` (Forward-Looking Planning Draft) |
| **Run ID / Timestamp** | `YYYYMMDD_HHMMSS` *(upcoming execution run; baseline reference run: `20261021_121434`)* |
| **Author / Owner** | `[ANALYST: name, role]` |
| **Project Scope** | IFRS 9 Expected Credit Loss (ECL) Methodology Demonstration (Public Data) |
| **Regulatory Alignment References** | IFRS 9, OSFI E-23 (§4.2 data quality & lineage), SR 26-2, ECOA *(illustrative/defensibility alignment; not a compliance certification)* |
| **Cross-Referenced Artifacts** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `FM-FDL-001` (A04), `FM-DCR-001` (A07), `FM-CFR-001` |

> **Governance & Scope Notice:** This document specifies the data provenance, sampling, missingness handling, leakage prevention, quality metrics, and fitness-for-purpose boundaries for the IFRS 9 ECL methodology demonstration on public Fannie Mae data. All unverified notebook outputs and schema items are explicitly marked for analyst verification rather than assumed.

---

## 1. Where does the data come from?

* **Source name:** Fannie Mae Single-Family Loan Performance Data
* **Publisher:** Federal National Mortgage Association
* **Access method:** Public download
* **Format:** ZIP archive containing CSV files (`Fannie Mae.zip` → `train4.csv`)
* **Total rows available:** `1,672,822`
* **Total columns available:** `133` *(matches the 133-column `train4.csv` header observed in sample inspection)*
* **Temporal coverage:** Vintage years `2021–2023` *(verify against full `YEARMONTH` / origination range in `train4.csv`)*
* **Licence:** Public dataset; terms permit analytical use *(verify specific Fannie Mae / Kaggle distribution terms)*

## 2. How did it get from source to analysis?

### 2.1 Source Publication

| Field | Value |
| :--- | :--- |
| **Dataset name** | Fannie Mae Single-Family Loan Performance |
| **Publisher** | Fannie Mae |
| **Publication form** | Public data tape |
| **Original file** | `Fannie Mae.zip` → `train4.csv` |
| **Original row count** | `1,672,822` rows |
| **Original column count** | `133` columns *(verified from sample header[cite: 1]; confirm in NB01 raw load log)* |
| **Vintage coverage** | `2021–2023` *(verify exact min/max `YEARMONTH` and origination dates in NB01)* |
| **License / terms** | Public dataset; terms permit non-commercial / analytical demonstration use *(verify exact publication terms)* |

### 2.2 Sampling

| Field | Value |
| :--- | :--- |
| **Sample size** | `100,000` rows |
| **Sampling method** | Random sample (unstratified, without replacement) |
| **Random seed** | `42` (project-wide; specified in `AS-PR004` §8) |
| **Sampling rationale** | Computational tractability without material distributional distortion |
| **Sampling reproducibility** | Same seed + same source file → same sample. The sample's content hash is the reproducibility check. |

* **Note on sampling:** The sample is random, not stratified. For the ECL demonstration pipeline, the observed sample target rate (`0.94–0.95%`) matches the full-file rate to two decimal places. Note per `L11` (`FM-PDR-001` §4) that row-level sampling on a loan-month panel samples loan-month observations rather than distinct loan IDs unless deduplicated upstream.

### 2.3 Load Path

The sample is loaded in **NB01 Phase 2** using a two-location file resolution:
* **Path 1:** `train4.csv` (working directory) *(verify against NB01 Phase 2 code if `loans_full_schema.csv` is still referenced as an alias)*
* **Path 2:** `data/train4.csv` (data folder)

If neither file exists, NB01 raises `FileNotFoundError` and halts immediately. The **no-synthetic-fallback rule** applies: a missing source file stops execution and never triggers a fallback to synthetic data. The audit log records the source file's hash so the exact input file is verifiable from the run.

---

### 3.1 Pipeline Lineage

Fannie Mae.zip → train4.csv (1,672,822 rows × 133 cols)
       │
       ▼
NB01 Phase 2: random sample (seed 42) → 100,000 rows
       │
       ▼
NB01 Phase 3–6: target verification check, missingness analysis,
                leakage detection, temporal audit, feature inventory
       │
       ▼
fannie_mae_final_clean.csv (100,000 × 175)
       │  content hash: bc5263ca06d29037
       │  (baseline run: 20261021_121434)
       ▼
NB02–NB06: downstream pipeline
       │
       ▼
NB03 parameters: outputs/notebook_03_parameters.json
NB04 results:    outputs/notebook_04_results.json

---

### 3.2 Source-System Lineage

The lineage inside Fannie Mae's publication process—and how the Kaggle derivative `train4.csv` constructed rolling windows, `DPD_INT`, and `TARGET` from raw Fannie Mae loan performance tapes[cite: 1]—is not recorded inside the pipeline.

### 3.3 Lineage Verification

| Check | How to Verify |
| :--- | :--- |
| **Clean frame shape (`100,000 × 175`)** | `fannie_mae_final_clean.csv` header and row count *(reconcile vs. `179` in NB01/NB02)* |
| **Clean frame content hash** | Recorded in the audit log (`bc5263ca06d29037` for run `20261021_121434`) |
| **Sample drawn from source** | NB01 audit log records the source file hash |
| **Downstream contract integrity** | NB02–NB06 assert the upstream contract's presence and reconcile run IDs (`YYYYMMDD_HHMMSS`) |

Because source-system lineage is external to the pipeline, it is recorded as a data limitation in **§9 (`D1`, `L11`, `L12`)**.

---

## 4. What is missing, and how was it handled?

### 4.1 Summary

| Field | Value |
| :--- | :--- |
| **Total missing values** | `[ANALYST: insert exact count from NB01 Phase 4 output on train4.csv]` |
| **Columns with missing values** | `10` *(verify against NB01 Phase 4 output on `train4.csv`)* |
| **Columns above 20% missing** | `5` *(verify against NB01 Phase 4 output on `train4.csv`)* |
| **Missingness threshold (high-risk)** | `20%` (per DCR data-quality rule) |
| **Imputation method** | Median (numeric) / mode (categorical) |
| **Missing indicators** | Created before imputation; preserved as `*_missing` columns |
| **Post-imputation missingness** | Zero missing values in the clean frame |

### 4.2 High-Missingness Columns (`train4.csv`)

*(Replace any non-Fannie Mae column names with exact NB01 Phase 4 outputs on `train4.csv`; note that `train4.csv` also uses sentinel values such as `999` in `MTH_SINCE_LST_DPD` and `MTH_SINCE_LAST_UPB_INC`[cite: 1] that must be checked alongside `NaN` missingness.)*

| Column (`train4.csv`) | Missing Count | Missing % | Structural / Domain Reason |
| :--- | :--- | :--- | :--- |
| `[ANALYST: insert column 1 from NB01 Phase 4]` | `[ANALYST: insert]` | `[ANALYST: insert %]` | `[ANALYST: e.g., co-borrower field FICO_CO_BOR[cite: 1] or joint-borrower attribute]` |
| `[ANALYST: insert column 2 from NB01 Phase 4]` | `[ANALYST: insert]` | `[ANALYST: insert %]` | `[ANALYST: insert structural reason]` |
| `[ANALYST: insert column 3 from NB01 Phase 4]` | `[ANALYST: insert]` | `[ANALYST: insert %]` | `[ANALYST: insert structural reason]` |
| `[ANALYST: insert column 4 from NB01 Phase 4]` | `[ANALYST: insert]` | `[ANALYST: insert %]` | `[ANALYST: insert structural reason]` |
| `[ANALYST: insert column 5 from NB01 Phase 4]` | `[ANALYST: insert]` | `[ANALYST: insert %]` | `[ANALYST: insert structural reason]` |

### 4.3 Why Indicators Are Created Before Imputation

The pipeline creates a `*_missing` flag for every column with missing values **before** imputation. This preserves the signal that a value was absent; imputing first and flagging afterward would destroy that information. Together with column-level missingness counts and documented median/mode rules, this implements the OSFI E-23 §4.2 data-quality traceability control.

### 4.4 What This Means for a Reader

Zero missing values in the clean frame means missingness was **handled**, not that it was harmless. A reader should inspect `*_missing` flags to see where imputation occurred. Furthermore, independent median/mode imputation does not preserve cross-column relationships (for example, between primary and co-borrower fields), which is documented as limitation **`D3`** in **§9.2**.

---

## 5. How was the sample drawn?

* **Method:** Simple random sample without replacement
* **Sample size:** `100,000` rows
* **Random seed:** `42`
* **Sampling fraction:** `5.98%` (`100,000 / 1,672,822`)

---

## 6. How are features screened and split for development and validation?

### 6.1 Feature Decisions (`data/feature_inventory.csv`)

The feature inventory (`data/feature_inventory.csv`) records, for every column in the clean frame:
* Data type
* Missing percentage
* Simplified IV approximation (`|corr| × 0.3`, capped at `0.5` — a univariate linear screening heuristic, not a textbook WOE-based IV)
* Inclusion decision (`INCLUDE` / `EXCLUDE`)
* Exclusion reason

This inventory acts as the schema contract between `NB01` and `NB02`. Because the IV column captures only univariate linear correlation, it must not be cited as evidence of multivariate feature strength.

#### What the Inventory Does Not Record
* **Feature interactions:** Univariate IV does not capture joint predictive relationships (`D2`).
* **Categorical encodings:** Encoding choices are modeling decisions documented in `NB02` and `FM-FDL-001` (A04).
* **Full narrative rationale:** `feature_inventory.csv` holds a one-line `Exclusion_Reason`; full rationale resides in `FM-FDL-001` (A04).

### 6.2 Development / Validation Split Structure

* `[ANALYST: insert split specification from NB02/NB03 — e.g., whether the IFRS 9 ECL demonstration runs across the full 100,000-row sample with deterministic parameter formulas, or uses a specific train/validation split and seed]`

---

## 7. Leakage Prevention and Diagnostics

### 7.1 Pre-Registered Leakage List (`KNOWN_LEAKAGE`)

Before data inspection, the pipeline freezes a pre-registered leakage list (`KNOWN_LEAKAGE`, governed in `FM-FDL-001` / A04) identifying post-origination, servicing-outcome, and target-derived columns in `train4.csv`[cite: 1] that must not enter point-in-time / origination feature matrices without explicit governance approval:

* `[ANALYST: insert exact Fannie Mae KNOWN_LEAKAGE column list from FM-FDL-001 / NB01 Phase 5 — e.g., TARGET, DPD_INT, DPD_*, UBP_*, CUR_UBP, CUR_IR_RAW, CUR_IR_SP*, MOD_FLAG, SER_FLAG, MTH_SINCE_LST_DPD, MTH_SINCE_LAST_UPB_INC[cite: 1], plus target transforms delinquency_state, delinquency_label, ifrs9_stage, default_flag]`

### 7.2 Why Pre-Registered

A post-hoc correlation scan alone cannot distinguish true causal risk drivers from post-outcome leakage. Freezing `KNOWN_LEAKAGE` prior to inspection ensures leakage exclusions are rule-driven rather than tuned to observed model metrics.

### 7.3 Diagnostic Leakage Scan (NB01 Phase 5)

NB01 Phase 5 runs two corroborative diagnostic checks:
* **Test A:** Univariate correlation with `TARGET`[cite: 1] `> 0.5` (`HIGH` severity if `> 0.6`, `MEDIUM` if `0.5–0.6`).
* **Test B:** Missingness–`TARGET`[cite: 1] correlation `> 0.15` (`HIGH` severity if `> 0.25`, `MEDIUM` if `0.15–0.25`).

Features flagged by the scan that are not already in `KNOWN_LEAKAGE` are routed to `FM-FDL-001` (A04) for analyst review.

### 7.4 Leakage Verification Outcome (Fannie Mae Pipeline)

* `[ANALYST: insert NB01 Phase 5 / NB02 leakage verification results on train4.csv — confirm all pre-registered leakage columns were excluded and record any diagnostic flags or findings]`

---

## 8. Data Quality Score

The pipeline computes a data quality score of `58.9 / 100` (`POOR`) from four penalty components *(verify exact component breakdown against NB01 Phase 4/6 log on `train4.csv`)*:

| Penalty | Contribution | What It Measures |
| :--- | :--- | :--- |
| **Missingness** | `21.1` | Average missingness across columns with missing values, halved, capped at `30` |
| **Duplicates** | `[ANALYST: insert from NB01 output; 0 if zero duplicates]` | Duplicate rows in the clean frame |
| **Leakage** | `15.0` | `3` per leakage feature, capped at `15` |
| **Macro gap** | `5.0` | Absence of macroeconomic time-series indicators |

The score's bands (`GOOD ≥ 80`, `MODERATE ≥ 60`, `POOR ≥ 40`, `CRITICAL < 40`) are descriptive conventions rather than DCR pass/fail thresholds. A `POOR` (`58.9`) score signals known structural properties of the public dataset (high structural missingness on optional fields, post-origination columns present in the raw tape before exclusion, and lack of macro variables) and should be read alongside **§9**.

---

## 9. Limitations and Fitness for Purpose

### 9.1 Model & Data Limitations (`L1`–`L12`)

| ID | Limitation | Impact | Source |
| :--- | :--- | :--- | :--- |
| **L1** | PD–LGD correlation is constructed | Convexity illustrative, not empirical | `notebook_03_limitations.csv` |
| **L2** | SICR uses top-decile proxy | Stage 2 is illustrative, not observed SICR | `notebook_03_limitations.csv` |
| **L3** | LGD is a function of PD | Average LGD `44.28%` (inside benchmark) | `notebook_03_limitations.csv` |
| **L4** | Prepayment/curtailment are assumptions | EAD approximate | `notebook_03_limitations.csv` |
| **L5** | EIR is industry average | Discounting approximate | `notebook_03_limitations.csv` |
| **L6** | PD cap masks tail convexity | Tail ECL understated pre-cap | `notebook_03_limitations.csv` |
| **L7** | Fairness testing incomplete | No direct protected attributes | `notebook_04_limitations.csv` |
| **L8** | U.S. data as Canadian proxy | Results illustrative | `notebook_04_limitations.csv` |
| **L9** | Scenario weights/multipliers assumption-based | Scenario severity directional | `notebook_04_limitations.csv` |
| **L10** | Capital coverage is diagnostic, not adequacy | CET1 is a ratio of RWA, not assets | `notebook_04_limitations.csv` |
| **L11** | Unit of analysis unverified (loan-month vs. distinct loan) | Portfolio counts and dollar totals conditional on row-level assumption | `FM-PDR-001` §4; `FM-CFR-001` |
| **L12** | Target construction (`TARGET` / `DPD_INT`) unverified | Default rate and IFRS 9 stage mapping conditional on undocumented Kaggle derivation | `FM-PDR-001` §5; `FM-CFR-001` |

### 9.2 Data-Specific Limitations (`D1`–`D4`)

| ID | Limitation | Why |
| :--- | :--- | :--- |
| **D1** | Source-system lineage not recorded | The pipeline records sample provenance (`train4.csv` hash) but not Fannie Mae / Kaggle upstream transformation logic[cite: 1] |
| **D2** | IV column is a simplified approximation | Univariate correlation heuristic (`|corr| × 0.3`) does not capture nonlinearities or feature interactions |
| **D3** | Joint/co-borrower imputation is independent | Median/mode imputation does not preserve conditional relationships across related columns |
| **D4** | Duplicate count not independently verified at loan-ID level | Row-level duplicate penalty is `[ANALYST: insert]`, and panel deduplication by `ID` × `YEARMONTH`[cite: 1] was not separately audited |

### 9.3 Fitness-for-Purpose Determination

* **Analyst Determination:** **Fit for IFRS 9 ECL methodology demonstration with documented caveats (`L1`–`L12`, `D1`–`D4`); not fit for production allowance recognition, regulatory capital calculation, or borrower-level decisions.**
* **Rationale:** The dataset supports demonstrating an end-to-end, reproducible IFRS 9 ECL workflow (`PD`, `LGD`, `EAD`, staging proxies, four-scenario probability weighting, and automated register guards) with explicit audit trails. Because the data lacks observed post-origination credit bureau updates, empirical recovery cash flows, macroeconomic indicators, Canadian mortgage loans, direct protected attributes, and verified `TARGET`/`ID` panel lineage (`L11`–`L12`), it does not support production provisioning or regulatory compliance claims.

### 9.4 What the Data Would Need to Support a Production Implementation

A production IFRS 9 ECL implementation would require:
* **Verified loan-level and loan-month lineage** — resolving `L11` and `L12` with documented delinquency and zero-balance definitions.
* **Current credit characteristics** — replacing the SICR top-decile proxy (`L2`) with observed borrower/account risk migration.
* **Empirical workout and recovery data** — calibrating LGD from realized post-default cash flows rather than rule-based formulas (`L3`).
* **Macroeconomic indicators** — linking scenario ECL shifts to validated macro drivers rather than analyst-specified multipliers (`L9`).
* **Canadian portfolio data** — validating parameters on target-jurisdiction mortgages (`L8`).
* **Protected attributes** — conducting direct fair-lending / disparate-impact testing rather than geographic-proxy checks (`L7`).
* **Out-of-time / walk-forward cohorts** — replacing an unstratified random row sample with time-ordered validation cohorts.

---

## 10. Data Governance and Access

### 10.1 Data Classification

| Field | Value |
| :--- | :--- |
| **Classification** | Public — Fannie Mae Single-Family Loan Performance is a public dataset |
| **Sensitive attributes** | None. The dataset contains no direct protected-class attributes and no direct borrower names/SSNs. |
| **Geographic granularity** | State (`PROP_STATE`), Metropolitan Statistical Area (`MSA`), and 3-digit ZIP (`ZIP`) are present in `train4.csv`[cite: 1] and are used solely as geographic proxies for exploratory fairness checks (`L7`), not as borrower identifiers. |
| **Re-identification risk** | Low. Public loan-performance tape with anonymized loan `ID` and coarse geography[cite: 1]. |

### 10.2 Access Controls

Not applicable — the data is public. Execution integrity is governed by the notebook environment, file hash verification, and run-ID logging (`YYYYMMDD_HHMMSS`).

### 10.3 Retention and Deletion

Not applicable — the data is public. The clean frame (`fannie_mae_final_clean.csv`), audit logs, manifests, and content hashes are retained in the repository for third-party reproducibility.

---

## 11. Document Control

| Field | Value |
| :--- | :--- |
| **Document ID** | `MAP-DS-FM-001` |
| **Version** | `0.2.0-draft` |
| **Status** | Draft — pending analyst insertion of `[ANALYST: insert ...]` NB01 metrics |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | `0.1` |
| **Related documents** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `FM-FDL-001` (A04), `FM-DCR-001` (A07), `FM-CFR-001`, `../powerbi_exports/data_dictionary.md` |
| **Next review** | Upon completion of `[ANALYST: insert ...]` NB01 fields or next notebook version bump |
