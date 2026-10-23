# Fannie Mae IFRS 9 ECL Demonstration — Analytical Specification

**Document ID:** FM-AS-001  
**Version / Status:** 1.4 / Frozen at NB03 v1.3 execution (run_id `20261021_121550`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Analytical Specification (AS)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · FM-AR-001 · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Specify what the Fannie Mae analysis must produce — artifacts, schemas, acceptance criteria, and the pipeline that produces them |
| **Notebooks specified** | NB01–NB06 |
| **Central contract** | `notebook_03_parameters.json` — the single frozen contract for NB04, NB05, and NB06 |
| **Reproducibility requirements** | Seed 42 (used for both sample draw and cross-validation); content hash; per-notebook manifest; per-run audit log; `run_id` and `notebook_version` on every artifact |
| **Where results live** | FM-DCR-001 §4 (measured values against criteria) and MAP-STF-FM-001 §7 (framework observations) — **not in this specification** |

> **Document purpose.** This specification defines what the Fannie Mae analysis must produce: the artifacts, their schemas, their acceptance criteria, and the pipeline that produces them. It is the specification against which the notebooks are validated.
>
> **On the relationship between this specification and the results.** The AS states what the artifacts must contain and what criteria they must satisfy. **The measured results** — whether the criteria were met, and what values the artifacts contain — are in FM-DCR-001 §4 (measured thresholds) and MAP-STF-FM-001 §7 (framework observations). The AS does not contain the current run's values; the analysis reports them.

---

## 1. Purpose and scope

This specification defines what the Fannie Mae analysis must produce: the artifacts, their schemas, their acceptance criteria, and the pipeline that produces them. It is the specification against which the notebooks are validated.

**In scope:** six notebooks (NB01–NB06), their outputs, the contracts between them, and the acceptance criteria.

**Out of scope:** the MAP documents, the production ECL implementation (which does not exist), and the Power BI dashboard's own specification (which is in NB06's own spec).

---

## 2. Stage specification

The pipeline has six stages, each implemented by one notebook. This section specifies, for each stage: the questions it answers, the inputs it requires, the outputs it produces, and the acceptance criteria its outputs must satisfy.

**This is the specification's core content.** A developer reading this section knows what to build; a validator reading it knows what to check. A reader who wants to know "what must NB03 produce?" finds the answer in one row of the table below.

### 2.1 — Stage overview

| Stage | Notebook | Questions answered | Inputs | Outputs | Key contract for next stage |
|---|---|---|---|---|---|
| 1 | NB01 | Is the data clean? What is the parameter construction? | `Fannie Mae.zip` → `train4.csv` | Clean dataframe, `pd_estimate`, `lgd_final`, `ead_final`, assumptions register, manifest | Clean data (`fannie_mae_final_clean.csv`) is NB02's input |
| 2 | NB02 | What is the risk factor analysis? | NB01 clean data | EDA artifacts (univariate stats, correlation, vintage, T-tests, 4 PNGs) | Descriptive only (no downstream contract) |
| 3 | NB03 | What are the IFRS 9 parameters (PD, LGD, EAD, stages, ECL)? | NB02 artifacts | `notebook_03_parameters.json`, limitations CSV | `notebook_03_parameters.json` is NB04–NB06's input |
| 4 | NB04 | What is the stress and fairness scenario analysis? | NB03 parameters | `notebook_04_results.json`, fairness limitations CSV | `notebook_04_results.json` is NB05's input |
| 5 | NB05 | Is the methodology compliant? | NB03, NB04 artifacts | Compliance report, decision log, manifest | Terminal (no downstream) |
| 6 | NB06 | What is the Power BI export? | NB03, NB04 artifacts | 9 exported files + manifest | Terminal (no downstream) |

**On the ordering.** NB01 must complete before NB02 (data quality feeds EDA). NB02 must complete before NB03 (EDA feeds parameter calibration). NB03 must complete before NB04 (parameters feed scenario analysis). NB04 must complete before NB05 (scenarios feed compliance). NB05 and NB06 are **terminal** — they read NB03 and NB04's artifacts and produce reports or exports, but they do not feed any downstream notebook.

**On the contract pattern.** Every stage writes a machine-readable contract that the next stage reads. The contract is the specification's enforcement mechanism: a missing or malformed contract raises `FileNotFoundError` or a validation error at the next stage's start, so the pipeline fails loudly at the first broken contract rather than silently continuing with stale data.

**The central contract.** `notebook_03_parameters.json` is the single frozen contract for the whole downstream pipeline. NB04, NB05, and NB06 all read it. **Changing it invalidates all downstream artifacts** — which is why the parameter JSON is the pipeline's reproducibility anchor.

### 2.2 — Acceptance criteria

The acceptance criteria for each stage are stated below. **The measured results against each criterion are in FM-DCR-001 §4 (results against criteria); this section states the criteria only.**

**NB01 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Content hash recorded | The `final_clean_content_hash` must be recorded in the manifest and in the cleaning documentation |
| Leakage count from inventory | The leakage count must be derived from the feature inventory, not hardcoded |
| Assumptions register | All parameters used in the pipeline must be recorded in `data/assumptions_register.csv` |
| Clean data shape | The cleaned dataframe must have 100,000 rows and 175 columns (after construction and missing-indicators) |
| Manifest completeness | Every saved artifact must be listed in `outputs/manifest_notebook_01.csv` with its SHA-256-16 hash |

**NB02 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Descriptive only | NB02 must not write a verdict, a risk rating, or a recommendation |
| Univariate stats | Every key risk driver must have descriptive statistics (count, mean, median, std, quartiles, skew, kurtosis) |
| Correlation | Both Pearson and Spearman correlations must be reported for the key risk drivers |
| Vintage analysis | If `YEARMONTH` is present, vintage cohorts must be computed; if not, the analysis must be explicitly skipped |
| T-tests | Welch's t-test must be used (unequal variances), not Student's t-test |

**NB03 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Parameter freezing | `IFRS9Config` must contain every parameter; `notebook_03_parameters.json` must contain the config and the computed results |
| **CCF = 0 asserted at three boundaries** | The CCF must be asserted to 0.0 at config creation, computation, and return (see FM-MD-001 §4.2) |
| Register guard behavior | AP-02 (LGD band), AP-03 (EAD ratio), AP-04 (ECL ratio) must fire as specified |
| Staging documented | Stage 1/2/3 fractions must be computed and documented, with the SICR proxy and C6 limitations disclosed |
| ECL computable | Per-stage ECL and probability-weighted ECL must both be computed |

**NB04 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Contract reconciliation | `upstream_nb03_run_id` must match NB03's `run_id` (contamination guard) |
| Scenario ECLs | All four scenarios must have PD, LGD, ECL, weight, and weighted ECL |
| Weight sum | The four scenario weights must sum to 1.0 |
| PD cap sensitivity | The `pd_cap_sensitivity` section must quantify the pre-cap tail understatement |
| Severity check | The historical comparison must be computed and labelled directional |
| Fairness scoping | The fairness limitations CSV must state that fairness is out of scope and why |

**NB05 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Claim-authorization self-check | The compliance report must contain no `Compliant`, `Non-Compliant`, `PASS`, `FAIL`, `Adequate`, `Inadequate`, or `Validated` tokens |
| Decision log | All decisions (D001–D025) must be documented with rationale and regulatory reference |
| Assumptions narrative | The narrative must be generated from the assumptions CSV (not hand-written) |
| Manifest scoping | The manifest must use scoped globs, not a full filesystem walk |

**NB06 acceptance criteria:**

| Criterion | Requirement |
|---|---|
| Promotion assertion | The final EXPORT_DIR must contain exactly `SPEC_EXPORTS` + `manifest.csv` (no stale artifacts) |
| C5 scope guard | The notebook must contain no ROC/AUC function in its namespace |
| Fact table correctness | `fact_loan_portfolio.csv` must have 100,000 rows; `fact_scenario_analysis.csv` must have 4 rows; `fact_ifrs9_stages.csv` must have 3 rows |
| Data dictionary | Every column in every export must be documented in `data_dictionary.md` |
| Neutral labels | Fairness status must be neutral (`TESTED` / `NOT ASSESSED`), not `PASS` / `FAIL` |

---

## 3. Artifact schemas

This section specifies each artifact's schema — the columns or fields it must contain, their types, and their required populated state. **This is what a developer implements against.** The current values in each artifact are not specified here; the schema is.

### 3.1 — NB01 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `data/assumptions_register.csv` | CSV | `Parameter`, `Value`, `Rationale`, `Regulatory_Reference`, `Notebook`, `Section`, `run_id`, `notebook_version` | One row per assumption; all fields populated |
| `data/fannie_mae_raw_sample.csv` | CSV | Same columns as source (175 after construction) | 100,000 rows; raw sample preserved separately from clean data |
| `data/fannie_mae_final_clean.csv` | CSV | 175 columns | Clean dataframe; content hash recorded |
| `outputs/manifest_notebook_01.csv` | CSV | `file`, `hash_sha256_16`, `run_id`, `notebook_version`, `timestamp` | Every saved artifact listed with SHA-256-16 hash |
| `outputs/data_cleaning_documentation.txt` | text | Cleaning decisions, limitations L1–L5 | All decisions documented; limitations listed |

### 3.2 — NB02 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `outputs/nb02_univariate_stats.csv` | CSV | 13 stats × 6 variables | Every key risk driver has descriptive statistics |
| `outputs/nb02_outlier_summary.csv` | CSV | 6 fields × 5 variables | Outlier bounds documented for each variable |
| `outputs/nb02_segment_analysis.csv` | CSV | Segment reports | Multiple segmentations (property type, occupancy status) |
| `outputs/nb02_correlation_comparison.csv` | CSV | `Variable`, `Pearson`, `Spearman`, `Difference` | Both correlation methods reported |
| `outputs/nb02_vintage_yearly.csv` | CSV | Vintage stats | Yearly cohorts with default rate and average age |
| `outputs/nb02_ttest_results.csv` | CSV | Welch's t-test results | 5 variables with t-stat, p-value, means, difference |
| 4 PNGs, 3 TXT docs | various | Descriptive | All produced |

### 3.3 — NB03 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `outputs/notebook_03_parameters.json` | JSON | `config` (all `IFRS9Config` values), `results` (PD, LGD, EAD, stage distribution, weighted ECL), `run_id`, `notebook_version` | CCF = 0 asserted; all fields populated |
| `outputs/notebook_03_limitations.csv` | CSV | L1–L6 methodology limitations (the full L1–L10 list is in FM-CFR-001 §3.3; the CSV is a subset relevant to NB03) | All limitations present with ID, description, impact, mitigation, regulatory reference |
| `outputs/pd_lgd_ead_calibration_summary_v1_3.txt` | text | Calibration summary | Evidence-not-declaration guardrail (no compliance verdict tokens) |

**On the limitations CSV scope.** The `notebook_03_limitations.csv` contains L1–L6 (the methodology's parameter-related limitations). The full L1–L10 list is in FM-CFR-001 §3.3. **The CSV is a subset, not the complete list** — a reader who wants the complete limitations reads the CFR.

### 3.4 — NB04 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `outputs/notebook_04_results.json` | JSON | `scenarios` (four scenarios), `total_weighted_ecl`, `ecl_ratio_of_ead`, `pd_cap_sensitivity`, `severity_check`, `fairness`, `upstream_nb03_run_id` | `upstream_nb03_run_id` matches NB03's `run_id` (contamination guard) |
| `outputs/notebook_04_fairness_limitations.csv` | CSV | Fairness scoping | Fairness out-of-scope documented; no protected attributes available |

### 3.5 — NB05 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `outputs/assumptions_register_narrative.md` | Markdown | Narrative of assumptions | Generated from CSV (not hand-written) |
| `outputs/decision_log_v1_2.txt` | text | 25 decisions (D001–D025) | All 25 documented with rationale and regulatory reference |
| `outputs/ifrs9_compliance_report_v1_2.txt` | text | 7 sections | Claim-authorization self-check passes (no verdict tokens) |
| `outputs/manifest_pipeline_v1_2.csv` | CSV | Every analysis artifact, with SHA-256-16 hash | Scoped globs (no full filesystem walk) |

### 3.6 — NB06 outputs

| Artifact | Format | Schema | Content requirement |
|---|---|---|---|
| `powerbi_exports/fact_loan_portfolio.csv` | CSV | 100,000 rows | One row per loan |
| `powerbi_exports/fact_scenario_analysis.csv` | CSV | 4 rows (one per scenario) | Values match NB04 |
| `powerbi_exports/fact_fairness_testing.csv` | CSV | Neutral interpretation | No PASS/FAIL tokens; `UNRELIABLE` flag present for high-cardinality results |
| `powerbi_exports/summary_metrics.csv` | CSV | 10 metrics | Fairness status is neutral (`TESTED` / `NOT ASSESSED`) |
| `powerbi_exports/dim_segment.csv` | CSV | 5 rows | Segment descriptions |
| `powerbi_exports/data_dictionary.md` | Markdown | Full column documentation | Every column documented; limitations embedded |
| `powerbi_exports/manifest.csv` | CSV | 9 exports hashed | Exactly `SPEC_EXPORTS` present |

---

## 4. Contract enforcement

The contract chain is the specification's enforcement mechanism. Every stage asserts its upstream contracts at start; a missing or malformed contract raises an error before any analysis runs.

NB01 → data/assumptions_register.csv + data/fannie_mae_final_clean.csv → NB02
NB02 → EDA artifacts (descriptive) → NB03
NB03 → notebook_03_parameters.json (frozen contract) → NB04, NB05, NB06
NB04 → notebook_04_results.json → NB05
NB05 → compliance report → terminal
NB06 → Power BI exports → terminal


**The parameter JSON is the pipeline's central contract.** It is the single source of truth for every parameter value. **Changing it invalidates everything downstream** — the contract's `run_id` is compared by NB04's `upstream_nb03_run_id` assertion, and a mismatch halts the pipeline.

**Every stage asserts its upstream contracts before reading.** A missing contract raises `FileNotFoundError`; a malformed contract raises a validation error; a contamination (mismatched `run_id`) raises an assertion error. **The specification is enforced at runtime, not in prose.**

---

## 5. Reproduction and versioning

### 5.1 — Requirements

| Requirement | What the specification requires |
|---|---|
| Sample seed | The 100,000-row sample is drawn with `np.random.default_rng(42)` — deterministic and reproducible |
| Training seed | The cross-validation and any stochastic operations use `RANDOM_STATE = 42` — a single project-wide seed |
| Content hash | Each notebook logs a SHA-256-16 content hash of its final artifact, so a re-run can be compared to the prior run |
| Manifest | Each notebook writes a per-notebook manifest listing every artifact it produced, with hashes |
| Audit log | Each notebook writes a per-run audit log, timestamped and versioned |
| `run_id` | Every artifact carries a `run_id` (timestamp-based) |
| `notebook_version` | Every artifact carries a `notebook_version` (semantic version) — **the version is taken from the notebook header, not fixed in the AS** |

### 5.2 — On the seed convention

**The sample seed and the training seed are the same value (42), but they are used in two places.** The sample seed is used for the 100,000-row draw in NB01; the training seed is used for the cross-validation in NB02 and any stochastic operations downstream. **They are the same value because the project's convention is a single project-wide seed**, carried forward from the Lending Club pipeline. **They are not independent seeds that happen to have the same value** — they are the same seed used in two places.

**On changing the seed.** Changing `RANDOM_STATE` invalidates both the sample and the cross-validation. **The two must be changed together** if they are changed at all. The v1.3 AS said "the sample seed and training seed are the same value (42), but they're independent. Changing one doesn't affect the other." **That claim was incorrect** — the two uses of the seed are coupled because they are the same value. The v1.4 AS corrects this.

### 5.3 — On the content hash

**The content hash is the sample's fingerprint.** A re-run with the same seed, same data, and same cleaning logic produces the same content hash. A different content hash means the input data, the cleaning, or the code changed. **The content hash is what makes "did the sample change?" answerable across runs** — the file hash tells you the source was the same; the content hash tells you the cleaned dataframe was the same.

---

## 6. Open items and dependencies

### 6.1 — Open items (AS-specific)

These are items about the *specification itself* — gaps or imprecisions in the AS. They are distinct from findings (CFR) and from methodology limitations (MD).

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **The AS does not specify the schema for the `fairness` section in `notebook_04_results.json`** — it lists the field but not its sub-fields (`cardinality`, `di_ratio`, `interpretation` per attribute). | Documented here; a future AS revision would specify the sub-schema. | MINOR version bump |
| 2 | **The AS's §3.3 lists NB03's limitations CSV as L1–L6, but the CFR lists L1–L10** — the AS's subset is correct (L7–L10 are fairness, Canadian data, scenario, and capital limitations, not parameter limitations), but the AS does not say why the subset is L1–L6. | Documented in §3.3's note; a future AS revision could cite the CFR's full list. | MINOR version bump |
| 3 | **The AS does not specify the allowed values for `Interpretation` in `fact_fairness_testing.csv`** — the v1.3 AS said "neutral interpretation" without enumerating the values. | Documented here; a future AS revision would specify the allowed values (`UNRELIABLE — high cardinality (>50)`, `DI≥0.80`, `DI<0.80`). | MINOR version bump |

### 6.2 — Items in other documents (not AS open items)

The following items appeared in the v1.3 AS as "open items." They are not AS open items — they belong to other documents — but they are noted here for reference:

| v1.3 item | Actual location |
|---|---|
| "The AP-03 failure is expected" | This is a **result and an observation**, not an AS open item. It belongs in FM-DCR-001 §4 (the AP-03 result) and MAP-STF-FM-001 §7 (the framework observation). |
| "The L6 PD-cap understatement is quantified in NB04 (14.3%)" | The **quantification** is a result (in `notebook_04_results.json`); the **limitation** is in FM-DS-001 §8; the **observation** is in MAP-STF-FM-001 §7. |
| "The claim-authorization self-check passes" | This is a **result**, not an AS open item. It belongs in FM-DCR-001 §4 (the self-check result) and MAP-STF-FM-001 §7 (the claim-authorization observation). |

**On the distinction.** The v1.3 AS's "open items" were results and observations, not AS gaps. **The v1.4 AS's open items are schema gaps in the specification itself** — things the AS does not yet specify. This is the same discipline applied to the Lending Club AS revision: the AS's open items are about the specification, not about the analysis.

### 6.3 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | FM-PDR-001, FM-DS-001, FM-FDL-001, FM-MD-001 (all referenced by the AS) |
| Downstream | All six notebooks implement the AS; the Analysis Code & Results records what they produced |
| Reproducibility | AS version is recorded in every notebook's header |

---

*End of document.*
