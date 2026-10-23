# Fannie Mae IFRS 9 ECL Demonstration — Analysis Code and Results

**Document ID:** FM-ACR-001  
**Version / Status:** 1.4 / Frozen at NB06 v1.2 execution (run_id `20261021_121555`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Analysis Code and Results (ACR)  
**Source notebooks:** NB01 v1.2 · NB02 v1.2 · NB03 v1.3 · NB04 v1.2 · NB05 v1.2 · NB06 v1.2  
**Companion documents:** FM-AS-001 (Analytical Specification) · FM-DCR-001 (Decision Criteria Register) · FM-CFR-001 (Consolidated Findings Register) · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan) · FM-AR-001 (Approval Record)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Index what code was run and what it produced; evidence that the code matches the AS |
| **Notebooks indexed** | NB01 v1.2 · NB02 v1.2 · NB03 v1.3 · NB04 v1.2 · NB05 v1.2 · NB06 v1.2 |
| **Run IDs** | NB01 `20261021_121434` · NB02 `20261021_121539` · NB03 `20261021_121550` · NB04 `20261021_121552` · NB05 `20261021_121553` · NB06 `20261021_121555` |
| **Approximate code size** | ~4,200 lines across 6 notebooks (see §2.1 for measurement) |
| **Where the measured values live** | FM-DCR-001 §4 — **not in this index** |
| **Where the limitations live** | FM-CFR-001 §3 — **not in this index** |
| **Code defects** | None documented (see §7.3) |

> **Document purpose.** This deliverable records **what code was run and what it produced**. It indexes the six notebooks, records their versions and run IDs, and provides the evidence that the code matches the Analytical Specification (FM-AS-001).
>
> **What this document does not do.** It does not reproduce the code (the notebooks are the code). It does not re-derive the results (FM-DCR-001 §4 is the authoritative record of measured values). It does not record the limitations (FM-CFR-001 §3 is the authoritative register). **It is an index and a coverage check, not a parallel record of results or limitations.**

---

## 1. Purpose and scope

This deliverable records what code was run and what it produced. It indexes the six notebooks, records their versions and run IDs, summarizes their outputs, and provides the evidence that the code matches the Analytical Specification.

**The critical difference from Lending Club.** The Fannie Mae pipeline has **no verdict in the code**. The code produces parameters and artifacts; the verdict lives in the framework document (MAP-STF-FM-001) and the Methodology Owner's decision (FM-AR-001). **The code quality claim is about parameter defensibility and contract enforcement, not about verdict correctness** — there is no verdict in the code to be correct or incorrect.

**What it does not do:**

- It does not reproduce the code (the notebooks are the code).
- It does not re-derive the results (FM-DCR-001 §4 is the authoritative record).
- It does not record the limitations (FM-CFR-001 §3 is the authoritative register).
- It does not assess the methodology's defensibility (MAP-STF-FM-001 §7 is the authoritative interpretation).

**On the relationship between this document and the DCR/CFR.** The ACR's §3 summarizes **what the notebooks produced** — the artifacts and their names, the run IDs, and the shape of the output. The **values** in those artifacts are recorded in FM-DCR-001 §4, where they are compared against criteria. The **limitations** are recorded in FM-CFR-001 §3. **The ACR does not restate either**; it points to both.

---

## 2. Code inventory

### 2.1 — Notebooks

| Notebook | Version | Run ID | Purpose | Approximate lines |
|---|---|---|---|---|
| NB01 | 1.2 | `20261021_121434` | Data loading + parameter construction | ~600 |
| NB02 | 1.2 | `20261021_121539` | EDA (descriptive) | ~800 |
| NB03 | 1.3 | `20261021_121550` | PD/LGD/EAD calibration | ~700 |
| NB04 | 1.2 | `20261021_121552` | Stress + fairness | ~800 |
| NB05 | 1.2 | `20261021_121553` | Reporting & compliance | ~600 |
| NB06 | 1.2 | `20261021_121555` | Power BI export | ~700 |

**Total:** approximately 4,200 lines across 6 notebooks.

**On the line counts.** The counts are approximate. They were measured by counting non-blank lines in each notebook's `.ipynb` file, excluding cells that are pure markdown. **The counts are stated for orientation, not as a precise measurement** — a reviewer who wants an exact count can measure the notebooks directly.

### 2.2 — Notebook headers

Each notebook's header records:

- **Version number** — e.g., `NOTEBOOK_VERSION = "1.3"`
- **Random seed** — `RANDOM_STATE = 42`
- **List of edit markers** — `E-NXX-XX` entries documenting what changed and why
- **What breaks if changed** — the downstream contracts that the notebook's outputs satisfy
- **What the notebook does NOT do** — the scope boundaries

**The header is the code's specification summary.** A reviewer who reads only the header knows what the notebook is for, what it is not for, and what depends on it.

---

## 3. What the notebooks produced

This section records **what artifacts the notebooks produced** — the artifact names, the run IDs, and the shape of the output. **The measured values in those artifacts are in FM-DCR-001 §4; the limitations are in FM-CFR-001 §3.** The ACR does not restate either.

### 3.1 — NB01 (Data loading + parameter construction)

**Artifacts produced:** `data/assumptions_register.csv`, `data/fannie_mae_raw_sample.csv`, `data/fannie_mae_final_clean.csv`, `outputs/manifest_notebook_01.csv`, `outputs/data_cleaning_documentation.txt`.

**Where the values are:** FM-DCR-001 §4.1 (data quality results).

**Where the limitations are:** FM-CFR-001 §3.1 (L-01.01 through L-01.03, data-preparation limitations).

### 3.2 — NB02 (EDA, descriptive)

**Artifacts produced:** `outputs/nb02_univariate_stats.csv`, `outputs/nb02_outlier_summary.csv`, `outputs/nb02_segment_analysis.csv`, `outputs/nb02_correlation_comparison.csv`, `outputs/nb02_vintage_yearly.csv`, `outputs/nb02_ttest_results.csv`, plus four PNGs and three TXT documentation files.

**Where the values are:** FM-DCR-001 §4 does not have a specific section for EDA results — **NB02 produces descriptive statistics, not criteria-meeting results**. The EDA's contribution is the context for the parameter choices; a reader who wants the EDA results reads the CSVs directly.

**Where the limitations are:** FM-CFR-001 §3.2 (NB02 produces no findings).

**On NB02's role in the ACR.** NB02 is the one notebook whose output is not directly tied to a DCR criterion. It is descriptive, not evaluative. **The ACR's index role is more important for NB02 than for the other notebooks** — a reader who wants to know what NB02 produced must read the artifact list, because there is no DCR criterion to point to.

### 3.3 — NB03 (PD/LGD/EAD calibration)

**Artifacts produced:** `outputs/notebook_03_parameters.json`, `outputs/notebook_03_limitations.csv`, `outputs/pd_lgd_ead_calibration_summary_v1_3.txt`.

**Where the values are:** FM-DCR-001 §4.2 (parameter results), §4.3 (staging results).

**Where the limitations are:** FM-CFR-001 §3.3 (L1–L10).

### 3.4 — NB04 (Stress + fairness)

**Artifacts produced:** `outputs/notebook_04_results.json`, `outputs/notebook_04_fairness_limitations.csv`.

**Where the values are:** FM-DCR-001 §4.4 (scenario results).

**Where the limitations are:** FM-CFR-001 §3.3 (L6 PD-cap effect, L7 fairness scoping, L9 scenario assumptions).

### 3.5 — NB05 (Reporting & compliance)

**Artifacts produced:** `outputs/assumptions_register_narrative.md`, `outputs/decision_log_v1_2.txt`, `outputs/ifrs9_compliance_report_v1_2.txt`, `outputs/manifest_pipeline_v1_2.csv`.

**Where the values are:** FM-DCR-001 §4.6 (governance results).

**Where the limitations are:** FM-CFR-001 §3.3 (the limitations mapped to regulatory requirements).

### 3.6 — NB06 (Power BI export)

**Artifacts produced:** `powerbi_exports/fact_loan_portfolio.csv`, `fact_scenario_analysis.csv`, `fact_fairness_testing.csv`, `fact_ifrs9_stages.csv`, `fact_capital_coverage.csv`, `fact_segment_analysis.csv`, `summary_metrics.csv`, `dim_segment.csv`, `data_dictionary.md`, `manifest.csv`.

**Where the values are:** FM-DCR-001 §4 (the values exported are the same values measured against criteria).

**Where the limitations are:** The data dictionary (`powerbi_exports/data_dictionary.md`) embeds the limitations; FM-CFR-001 §3.3 is the authoritative register.

---

## 4. Code quality evidence

The code quality evidence is the patterns that repeat across notebooks. **Each pattern is cited to a code location** so a validator can verify the claim.

| Pattern | Where | Code location (evidence) |
|---|---|---|
| **Audit logger** | All notebooks | `setup_audit_logger(run_id=None)` — one per notebook, distinct log filename prefix (`ecl_`, `eda_`, `calibration_`, `stress_testing_`, `reporting_`, `powerbi_export_`) |
| **File hashing** | All notebooks | `compute_file_hash(path)` — SHA-256-16, chunked read (64 KB blocks), used for every saved artifact |
| **Save with provenance** | All notebooks | `_stamp_and_save` / `_write_json` / `_write_staged_csv` — attaches `run_id` and `notebook_version` to every artifact |
| **Contract assertion** | All notebooks | `REQUIRED = {...}` dict at notebook start; `if not Path(p).exists(): raise FileNotFoundError(...)` — no silent fallback |
| **Register guards (AP-02/03/04)** | NB03 | Two-tier warning/error branches in the LGD, EAD, and ECL calculators |
| **Claim-authorization self-check** | NB05 | Forbidden-token assertion in the compliance report writer |
| **Staging-then-promotion** | NB06 | Write to `_staging/`, then promote to `powerbi_exports/` with a `SPEC_EXPORTS` assertion |
| **C5 scope guard** | NB06 | `assert 'roc_curve' not in dir() and 'auc' not in dir()` |

**The patterns are the same as Lending Club's, but with different specifics.** The Fannie Mae pipeline adds four patterns that Lending Club does not have:

1. **Register guards (AP-02, AP-03, AP-04)** — parameter-property thresholds (LGD band, EAD ratio, ECL ratio) that fire on drift.
2. **Claim-authorization self-check (NB05)** — the forbidden-token assertion that prevents the compliance report from containing a verdict.
3. **Staging-then-promotion (NB06)** — a transactional file export that prevents stale artifacts.
4. **C5 scope guard (NB06)** — an assertion that no ROC/AUC function sneaks into a calibration-model export.

**Each pattern is a control.** The Fannie Mae pipeline has more controls because it is producing a **methodology that must be defensible to an IFRS 9 auditor**, not just a model that must be validated by a validator. **The extra controls reflect the different validation paradigm** — not a claim that Fannie Mae is "better controlled" than Lending Club, but a claim that the Fannie Mae pipeline's outputs are subject to a different kind of scrutiny.

---

## 5. Specification-to-code coverage

This section checks whether **every AS item has a code location**. The AS's §2 stage specification lists, for each notebook, the questions it answers and the acceptance criteria; §3 lists each artifact's schema. This section verifies that each of those items is implemented in the code.

**On the distinction from a mapping.** The v1.3 document called this section "Specification-to-code mapping" and presented a 16-row table. That was a **sample** of the mapping, not an enforcement. The v1.4 section is a **coverage check**: it enumerates the AS items and shows, for each, whether the code implements it.

### 5.1 — AS stage acceptance criteria (FM-AS-001 §2.2)

| AS item | Code location | Status |
|---|---|---|
| NB01: content hash recorded | NB01 Phase 5 (`final_clean_content_hash`) | Matched |
| NB01: leakage count from inventory | NB01 Phase 2 (leakage count derived from `feature_inventory.csv`) | Matched |
| NB01: assumptions register | NB01 Phase 1 (`assumptions_register.csv` writer) | Matched |
| NB01: clean data shape | NB01 Phase 5 (`fannie_mae_final_clean.csv` writer) | Matched |
| NB01: manifest completeness | NB01 Phase 5 (`manifest_notebook_01.csv` writer) | Matched |
| NB02: descriptive only (no verdict) | NB02 (no verdict field written) | Matched |
| NB02: univariate stats | NB02 Phase 1 (`nb02_univariate_stats.csv` writer) | Matched |
| NB02: correlation (Pearson + Spearman) | NB02 Phase 3 (`nb02_correlation_comparison.csv` writer) | Matched |
| NB02: vintage analysis (skip if YEARMONTH absent) | NB02 Phase 4 (`if 'YEARMONTH' in df.columns:` guard) | Matched |
| NB02: Welch's t-test | NB02 Phase 5 (`stats.ttest_ind(..., equal_var=False)`) | Matched |
| NB03: parameter freezing | NB03 Phase 1 (`IFRS9Config` dataclass) | Matched |
| NB03: **CCF = 0 asserted at three boundaries** | NB03 Phase 1, 5, 6 (three `assert config.ccf == 0.0`) | Matched |
| NB03: register guard behavior | NB03 Phase 4, 5, 6 (AP-02/03/04 branches) | Matched |
| NB03: staging documented | NB03 Phase 3 (stage distribution print; SICR proxy note) | Matched |
| NB03: ECL computable | NB03 Phase 6 (per-stage and weighted ECL) | Matched |
| NB04: contract reconciliation | NB04 Phase 2 (`assert nb04['upstream_nb03_run_id'] == nb03['run_id']`) | Matched |
| NB04: scenario ECLs | NB04 Phase 4 (`notebook_04_results.json` scenarios) | Matched |
| NB04: weight sum | NB04 Phase 4 (weights sum to 1.0 in scenario dict) | Matched |
| NB04: PD cap sensitivity | NB04 Phase 5 (`pd_cap_sensitivity` in JSON) | Matched |
| NB04: severity check | NB04 Phase 7 (`severity_check` in JSON) | Matched |
| NB04: fairness scoping | NB04 Phase 8 (`notebook_04_fairness_limitations.csv`) | Matched |
| NB05: claim-authorization self-check | NB05 Phase 7 (forbidden-token assertion) | Matched |
| NB05: decision log | NB05 Phase 4 (`decision_log_v1_2.txt` writer) | Matched |
| NB05: assumptions narrative | NB05 Phase 3 (`assumptions_register_narrative.md` writer) | Matched |
| NB05: manifest scoping | NB05 Phase 5 (scoped globs, not full filesystem walk) | Matched |
| NB06: promotion assertion | NB06 Phase 5 (`assert not missing; assert not extra`) | Matched |
| NB06: C5 scope guard | NB06 Phase 6 (`assert 'roc_curve' not in dir()`) | Matched |
| NB06: fact table correctness | NB06 Phase 2 (`fact_loan_portfolio.csv`, `fact_scenario_analysis.csv`, `fact_ifrs9_stages.csv` writers) | Matched |
| NB06: data dictionary | NB06 Phase 3 (`data_dictionary.md` writer) | Matched |
| NB06: neutral labels | NB06 Phase 2 (`fact_fairness_testing.csv` with neutral interpretation) | Matched |

**Coverage: 30 of 30 AS acceptance criteria have a code location.** All matched.

### 5.2 — AS artifact schemas (FM-AS-001 §3)

| AS artifact schema | Code location | Status |
|---|---|---|
| `assumptions_register.csv` schema | NB01 Phase 1 | Matched |
| `fannie_mae_raw_sample.csv` schema | NB01 Phase 2 | Matched |
| `fannie_mae_final_clean.csv` schema | NB01 Phase 5 | Matched |
| `manifest_notebook_01.csv` schema | NB01 Phase 5 | Matched |
| `data_cleaning_documentation.txt` structure | NB01 Phase 6 | Matched |
| `nb02_univariate_stats.csv` schema | NB02 Phase 1 | Matched |
| `nb02_outlier_summary.csv` schema | NB02 Phase 1 | Matched |
| `nb02_segment_analysis.csv` schema | NB02 Phase 2 | Matched |
| `nb02_correlation_comparison.csv` schema | NB02 Phase 3 | Matched |
| `nb02_vintage_yearly.csv` schema | NB02 Phase 4 | Matched |
| `nb02_ttest_results.csv` schema | NB02 Phase 5 | Matched |
| `notebook_03_parameters.json` schema | NB03 Phase 9 | Matched |
| `notebook_03_limitations.csv` schema | NB03 Phase 7 | Matched |
| `pd_lgd_ead_calibration_summary_v1_3.txt` structure | NB03 Phase 8 | Matched |
| `notebook_04_results.json` schema | NB04 Phase 11 | Matched |
| `notebook_04_fairness_limitations.csv` schema | NB04 Phase 9 | Matched |
| `assumptions_register_narrative.md` structure | NB05 Phase 3 | Matched |
| `decision_log_v1_2.txt` structure | NB05 Phase 4 | Matched |
| `ifrs9_compliance_report_v1_2.txt` structure | NB05 Phase 6 | Matched |
| `manifest_pipeline_v1_2.csv` schema | NB05 Phase 5 | Matched |
| Power BI fact tables (6) | NB06 Phase 2 | Matched |
| `summary_metrics.csv` schema | NB06 Phase 2 | Matched |
| `dim_segment.csv` schema | NB06 Phase 2 | Matched |
| `data_dictionary.md` structure | NB06 Phase 3 | Matched |
| `manifest.csv` schema | NB06 Phase 4 | Matched |

**Coverage: 25 of 25 AS artifact schemas have a code location.** All matched.

### 5.3 — Coverage summary

| Category | Items | Matched | Partial | Not found |
|---|---|---|---|---|
| AS acceptance criteria | 30 | 30 | 0 | 0 |
| AS artifact schemas | 25 | 25 | 0 | 0 |
| **Total** | **55** | **55** | **0** | **0** |

**The code matches the AS.** Every acceptance criterion and every artifact schema has a code location. The v1.3 document asserted this; the v1.4 document demonstrates it.

---

## 6. Reproducibility evidence

The reproducibility evidence is the artifacts that record what was run. **This section lists the artifacts; the artifacts themselves are the evidence.**

| Evidence | Location | Content |
|---|---|---|
| Run IDs | All notebooks | Timestamp-based, unique per execution |
| Content hash | `manifest_notebook_01.csv` | SHA-256-16 of final clean dataframe |
| File hashes | Manifests | SHA-256-16 of each artifact |
| Audit logs | `logs/*_audit_trail_*.log` | Per-run, per-notebook |
| Parameters JSON | `notebook_03_parameters.json` | Frozen contract |
| Limitations CSV | `notebook_03_limitations.csv` | L1–L6 documented |
| Power BI manifest | `powerbi_exports/manifest.csv` | 9 exports hashed |

**The parameters JSON is the reproducibility anchor.** Any reviewer can verify the frozen parameters and re-run the downstream pipeline from them. **The parameters JSON is the contract that makes the pipeline reproducible** — a re-run that produces the same JSON is a re-run that produces the same downstream artifacts.

**A reviewer can reproduce the results from the artifacts alone.** The artifacts are the reproduction. The notebooks are the code that produced them; the artifacts are the evidence that the code produced what the AS specified.

---

## 7. Code-level limitations and known defects

### 7.1 — On the distinction from methodology limitations

The methodology's limitations (L1–L10) are in **FM-CFR-001 §3.3**. The data limitations are in **FM-DS-001 §8**. The parameter values are in **FM-MD-001 §4**. **This section records only code-level limitations** — what the code does and does not do, independent of whether the methodology is sound.

### 7.2 — Code-level limitations

| ID | Limitation | Where | Impact |
|---|---|---|---|
| CL-1 | **Content hash is of the final clean dataframe, not the raw sample** | NB01 Phase 5 | A reviewer who wants to verify the raw sample independently must compute its hash separately — the manifest records both, but the content hash is the cleaner's fingerprint, not the sampler's |
| CL-2 | **The register guards log warnings but do not block execution** | NB03 Phase 4, 5, 6 | A guard that fires on a MAJOR change is a signal, not a gate — a run can complete with a fired guard, which is by design (the analyst investigates) but means a fired guard does not by itself halt the pipeline |
| CL-3 | **NB06's staging-then-promotion cleans stale artifacts but does not detect unexpected directories** | NB06 Phase 5 | If a subdirectory appears in `powerbi_exports/` (e.g., from a failed prior run), it is not removed by the cleanup loop, which only removes files. This is a minor gap in the "exactly `SPEC_EXPORTS`" assertion |

**On the distinction from methodology limitations.** The methodology's L6 (PD cap masks tail convexity) is a *methodology limitation* — it concerns what the ECL result can support. The ACR's CL-2 (register guards log warnings but do not block execution) is a *code limitation* — it concerns what the guard's enforcement does. They overlap in spirit but answer different questions.

### 7.3 — Known code defects

| Defect | Where | Impact | Status |
|---|---|---|---|
| None documented | — | — | — |

**The Fannie Mae pipeline has no known code defects.** This is the **"documentation, not correction" paradigm** — the pipeline is complete and correct, with documented limitations. **This is different from Lending Club**, which has one documented code defect (a missing `import warnings` in NB04) that was fixed at first run. **The difference is the paradigm**: Lending Club is a model development pipeline that went through a code defect fix; Fannie Mae is a methodology demonstration that was written more carefully from the start (the claim-authorization discipline is evidence of the care).

**On "no known defects" as a claim.** This is a claim about **what has been found**, not a claim that the code is defect-free. A code review could surface a defect tomorrow; the claim is that no defect is *currently documented*. **A reader who reads "None documented" and concludes "the code is perfect" has misread the claim** — the code is documented as defect-free, which is a statement about the documentation state, not a proof of correctness.

---

## 8. Open items and dependencies

### 8.1 — Open items (ACR-specific)

These are items about the *Analysis Code and Results document itself* — gaps or imprecisions. They are distinct from limitations (CFR) and from code-level limitations (this document's §7.2).

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **§2.1's line counts are approximate.** The counts were measured from the `.ipynb` files' non-blank lines (excluding markdown cells), but the measurement method is not auditable from this document alone. | Documented in §2.1; a future ACR revision could add the measurement script as an appendix. | MINOR version bump |
| 2 | **§5's coverage check lists 55 items; the AS may have additional items not captured in §5.1 and §5.2.** If the AS is revised to add items, this coverage table must be updated. | The coverage table is complete for the current AS (v1.4). A future AS revision would require an ACR update. | Follows the AS's version chain |
| 3 | **§7.2's code-level limitations list is not exhaustive.** There are likely other code-level limitations that have not been documented (e.g., the audit logger's log rotation behavior, the pickle file size constraints, the SHAP sample size). | Documented here as candidates for future ACR revisions. | MINOR version bump |

### 8.2 — Items in other documents (not ACR open items)

The following items appeared in the v1.3 ACR as "open items." They are not ACR open items — they belong to other documents — but they are noted here for reference:

| v1.3 item | Actual location |
|---|---|
| "The MAP Validation Opinion is pending" | This is a **pending validation lifecycle item**, not an ACR open item. It belongs in FM-RVP-001 §7.1 (Phase 6) and FM-AR-001 (the approval record). |
| "The production dependencies (P-LC-01 through P-LC-03) are documented in the Remediation Plan" | The **dependencies** are in MAP-RP-001; the **limitations** they address are in FM-CFR-001 §3.3. Not ACR open items. |
| "The AP-03 expected failure is documented in the AS and the Findings Register" | The **result** is in FM-DCR-001 §4.1; the **observation** is in MAP-STF-FM-001 §7; the **limitation** is in FM-CFR-001 §3.3. Not an ACR open item. |

**On the distinction.** The v1.3 ACR's "open items" were a mix of pending lifecycle items, dependencies, and results. **The v1.4 ACR's open items are gaps in the ACR itself** — things the ACR does not yet specify or enumerate. This is the same discipline applied to the Lending Club ACR revision: the ACR's open items are about the ACR, not about the analysis.

### 8.3 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | FM-AS-001 (the specification that this ACR checks coverage against); the six notebooks (the code being indexed) |
| Downstream | FM-CFR-001 (limitations), MAP-STF-FM-001 (framework observations), FM-AR-001 (approval decision) |
| Reproducibility | All artifacts carry `run_id` and `notebook_version` |

---

*End of document.*
