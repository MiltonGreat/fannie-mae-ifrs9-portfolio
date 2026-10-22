# Fannie Mae IFRS 9 ECL Framework

**A mortgage ECL stress-testing framework — and a case study in knowing what not to claim.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![IFRS 9](https://img.shields.io/badge/IFRS%209-Methodology%20Demo-orange)
![OSFI E-23](https://img.shields.io/badge/OSFI%20E--23-Aligned-lightgrey)
![SR 11-7](https://img.shields.io/badge/SR%2011--7-Partially%20Aligned-yellow)
![Status](https://img.shields.io/badge/Status-Portfolio%20Artifact-lightgrey)

---

## The business problem

A bank's finance and risk teams must estimate Expected Credit Losses (ECL) under IFRS 9 — a forward-looking, probability-weighted estimate of losses on the loan book, updated every reporting period. The estimate drives provisioning, capital planning, and stress testing.

The framework has three hard parts:

1. **PD, LGD, and EAD must be estimated** — not from historical averages, but from a model that responds to economic conditions.
2. **Loans must be sorted into three stages** (performing, significant increase in credit risk, credit-impaired), each with a different ECL basis.
3. **The result must be probability-weighted across scenarios** — baseline, adverse, severely adverse, and tail — and discounted to present value.

This project demonstrates all three on public Fannie Mae data, with every methodological decision documented. **What it does not do is calibrate PD, LGD, or EAD to a real bank's portfolio** — those are assumption-based, and the project is transparent about which assumption each number rests on.

**The framing discipline is the point.** An earlier version of this project characterized its capital result as "Inadequate under stress." That was wrong — the ratio was a scale mismatch, not a capital adequacy finding. Correcting that framing is the most valuable thing this project demonstrates about model governance.

**What the pipeline reports:**

| Metric | Value | What it means |
|---|---|---|
| Portfolio ECL as % of total assets (weighted) | 22.31% | A **coverage metric** — how much of the assumed asset base is consumed by expected losses |
| Portfolio ECL as % of total assets (tail) | 75.12% | Same, under the Tail Risk scenario |
| ECL / CET1 ratio | ~496% (weighted) | **Diagnostic only** — a scale-mismatched ratio, not a capital adequacy finding |

---

## What this framework demonstrates

- **A working IFRS 9 ECL pipeline:** PD/LGD/EAD calibration → three-stage classification → probability-weighted ECL across four scenarios
- **A documented decision trail:** 25 decisions with rationale and regulatory reference in the decision log
- **A data-quality discipline:** leakage, missingness, and scope limitations surfaced as findings, not suppressed
- **A clean separation between what the code computes and what the analyst declares:** every compliance status field is an `[ANALYST: insert status]` placeholder

## What this framework does NOT demonstrate

- **A production-calibrated ECL model** — parameters are assumption-based, not calibrated to a real bank's recovery data
- **Independent validation** — there is no second-line validation of this pipeline; the Validation Opinion is authored by the analyst, not by a validation notebook
- **Fair-lending compliance** — the Fannie Mae public dataset contains no protected-class attributes and no usable geographic proxies
- **Canadian applicability** — the pipeline is trained on U.S. data and is not validated for Canadian portfolios (OSFI E-23 scope restriction)
- **IFRS 9 compliance** — IFRS 9 compliance is a determination an institution makes against its regulatory obligations, not a claim a demonstration project can make for itself

**The last two are the most important.** A demonstration that over-claims is less useful than one that clearly names its limits.

---

## Key results

| Metric | Value | Notes |
|---|---|---|
| Portfolio size | 100,000 loans | Random sample from Fannie Mae dataset (seed 42) |
| Default rate (observed) | 0.94–0.95% | Backward-looking frequency in the sample |
| PD (calibrated, unconditional) | 0.95% | Sample default rate; clipped to floor 0.01%, cap 20% |
| Average LGD | 44.28% | Base 45% + age/LTV/property adjustments — **inside the 40–60% benchmark band** |
| Average EAD (per loan) | ~$204,745 | After prepayment (8%) and curtailment (2%) adjustments. **EAD ratio 86.95%, above the 50–85% target band — flagged as AP-03** |
| Probability-weighted ECL (per loan) | ~$2,231 | Across four scenarios (45/30/15/10 weights) |
| Portfolio ECL (weighted) | ~$223M | Per-loan ECL × 100,000 |
| ECL / total assets (coverage diagnostic) | 22.3% | **Not a capital-adequacy finding** — see "The framing correction" above |

> **Note on EAD ratio.** The EAD ratio of 86.95% is above the 50–85% target band. This triggers investigation trigger IT-03 and appears as Finding FM-R-08 in the remediation register. It is not necessarily a methodology error — a portfolio weighted toward younger loans (before prepayment and curtailment have had time to reduce the balance) will naturally show a higher ratio. It is disclosed and flagged, not suppressed.

---

## IFRS 9 parameters

| Parameter | Value | Description |
|---|---|---|
| Default rate (observed) | 0.94–0.95% | Backward-looking frequency in sample |
| PD (unconditional) | 0.95% | Forward-looking estimate; floor 0.01%, cap 20% |
| Average LGD | 44.28% | Base 45% + collateral adjustments; floor 10%, cap 90% |
| Average EAD | ~$204,745 | After prepayment (8%) and curtailment (2%) adjustments |
| EAD ratio (EAD / ORG_UPB) | 86.95% | Above 50–85% target band; flagged as AP-03 |
| ECL (unstressed, per loan) | ~$835 | PD × LGD × EAD × discount factor |
| Probability-weighted ECL (per loan) | ~$2,231 | Across four scenarios |
| Discount factor | 1 / (1 + 0.035) | EIR = 3.5% (industry average) |

### IFRS 9 stage distribution

| Stage | Share | ECL Basis |
|---|---|---|
| Stage 1 | ~89.1% | 12-month ECL |
| Stage 2 | ~9.9% | Lifetime ECL (proxy SICR) |
| Stage 3 | ~1.0% | Lifetime ECL (credit-impaired) |

**Stage 2 is SICR-by-construction, not observed credit deterioration.** The Stage 2 population is the highest-risk decile of non-defaulted loans by `pd_estimate`. This produces a Stage 2 fraction of ~9.9% by construction, not by observed credit deterioration. The mechanism is explicitly disclosed in the notebook's output, not hidden. Documented as Finding F-07 and Limitation L2.

**What this means for a reader:** do not read "9.9% Stage 2" as "9.9% of the portfolio had a significant increase in credit risk." It means "the top-decile proxy assigned 9.9% of loans to Stage 2." The distinction matters because a bank's reported Stage 2 fraction is an observed measurement; this project's is a construction.

---

## Scenario analysis

| Scenario | PD | LGD | ECL (per loan) | Weight | Weighted ECL |
|---|---|---|---|---|---|
| Baseline | 0.95% | 44.28% | ~$835 | 45% | ~$376 |
| Adverse | 1.72% | 53.13% | ~$1,803 | 30% | ~$541 |
| Severely Adverse | 2.86% | 66.41% | ~$3,756 | 15% | ~$563 |
| Tail Risk | 4.29% (capped) | 88.55% | ~$7,512 | 10% | ~$751 |

**Probability-weighted ECL (per loan): ~$2,231.**

### PD cap sensitivity

The Tail Risk scenario's PD is capped at 20% (the config's `pd_cap`). Without the cap, the pre-cap PD would be ~26.1%, and the uncapped ECL would be ~$16,858 per loan, versus $12,901 with the cap. The cap **understates Tail Risk ECL by ~30.7%**.

This is a deliberate stability measure, not a calibration error. It is documented as Limitation L6, and the uncapped value is reported alongside the capped value in NB04 so the effect is disclosed rather than hidden.

### Severity calibration check

The Severely Adverse ECL as a rate of EAD is compared against the 2008–2010 historical average (1.44%). The pipeline reports a ratio of ~1.27× historical — below the 1.5× "severely adverse" threshold in the Register.

This triggers investigation trigger **IT-05**. It is a directional check, not a reproduction of the 2008 crisis, and the notebook labels it as such. The comparison answers the question *"Is the scenario severe enough?"* It does not answer *"Would the portfolio have survived the 2008 crisis?"* — that would require the historical defaults themselves, not an average rate.

---

## Fairness testing

| Assessment | Status |
|---|---|
| Fair lending testing | **Not performed** |
| Reason | Dataset contains no protected-class attributes or usable geographic proxies |
| Attributes checked | race, ethnicity, sex, gender, marital status, national origin, religion, age band, state, region, census tract, ZIP, MSA |
| Attributes found | None usable as protected-class proxies |

**What this means.** Fair lending testing under ECOA (US), OSFI E-23 s.6.0 (Canada), and equivalent regimes requires testing on protected classes. The Fannie Mae public dataset does not contain them, and fair lending compliance **cannot be verified from this data**.

The pipeline **does** run fairness testing on the geographic proxies it has — state, MSA, ZIP — and reports the results with an explicit `UNRELIABLE` flag, because the 4/5ths rule at cardinality 52 / 328 / 638 produces degenerate results (many groups with zero defaults → DI = 0.00%, which is not evidence of discrimination). Documented as Finding FM-R-09 and Limitation F-L2.

**What a production fair lending assessment would require.** Data containing protected attributes or geographic identifiers (ZIP code, census tract, ideally census-tract-level demographics). For US mortgage portfolios, this is available through HMDA reporting. For Canadian portfolios, an equivalent disclosure regime would need to be identified.

---

## Regulatory compliance status

Each framework is assessed against what the project **actually demonstrates**, not what it would require in production.

| Regulation | Status | Notes |
|---|---|---|
| **IFRS 9** | Methodology demonstration only | ECL measurement, three-stage classification, and probability weighting implemented. SICR uses a proxy (top-decile, not true SICR). Not suitable for provisioning without recalibration. |
| **OSFI E-23** | Aligned with limitations | Data quality and documentation practices followed. Fair-lending testing not performed. Not validated for Canadian portfolios. |
| **SR 11-7** | Partially aligned | Model documentation and outcomes analysis implemented. **Independent validation was not performed** — the pipeline is a first-line methodology demonstration, not a second-line validation. |
| **ECOA** | Not assessed | Dataset lacks protected-class attributes. |
| **Basel III** | Framework reference only | Capital framework concepts (CET1, RWA) are referenced but the pipeline does not implement a capital adequacy calculation. Coverage diagnostic only. |

A full regulatory mapping — including the specific requirements addressed, the evidence, and the gaps — is in `outputs/reports/regulatory_mapping_v2.csv` and `outputs/reports/compliance_report_v2.txt`.

---

## The claim-authorization discipline

**This is the feature that most distinguishes the project.** Every notebook in this pipeline deliberately stops short of writing a compliance verdict. The claim-authorization discipline is enforced in code:

- **NB04** asserts that no fairness result contains a `PASS` token, and that high-cardinality results are flagged `UNRELIABLE`.
- **NB05** asserts that the executive summary is a placeholder and that the report contains no `Compliant`, `Non-Compliant`, `PASS`, `FAIL`, `Adequate`, `Inadequate`, or `Validated` tokens.
- **NB06** asserts that no ROC/AUC function is in the namespace (C5 scope guard — AUC does not apply to a calibration model).

**Why this matters.** The pipeline produces **evidence**, not **verdicts**. Every compliance status in the report is an `[ANALYST: insert status]` placeholder. The analyst writes the verdict; the notebook supplies the evidence. This is deliberate — it keeps the tool from over-claiming and keeps the human in the loop.

**The practical consequence.** A reader of this README who asks "is this IFRS 9 compliant?" will not find an answer here, because the project does not claim one. The answer belongs to the analyst, in the Validation Opinion — which is not in this repo, because this project is a framework, not a validated model.

---

## Notebooks overview

| Notebook | Focus | Key Outputs | Regulatory Link |
|---|---|---|---|
| 01 Data Loading | Quality & cleaning | Cleaned dataset (100,000 × 145), missing indicators, assumptions register, `data_quality_summary.json` | OSFI E-23 |
| 02 EDA | Risk factor analysis | Risk drivers, correlations, vintage patterns, t-test results | SR 11-7 (outcomes analysis) |
| 03 Calibration | PD/LGD/EAD calibration | IFRS 9 parameters, stage classification, LGD/EAD construction, ECL per stage and per scenario, `notebook_03_parameters.json` | IFRS 9 |
| 04 Stress Testing | Scenarios & coverage | Scenario ECLs, PD-cap sensitivity, severity check, coverage diagnostic, fairness assessment (UNRELIABLE at high cardinality) | IFRS 9, ECOA |
| 05 Reporting | Dashboard & report | Compliance report (with `[ANALYST: …]` placeholders), decision log, manifest | All |
| 06 Power BI Export | Data export | Fact and dimension tables, data dictionary, manifest | Reporting |

**Run the notebooks in order.** Each depends on artifacts produced by the preceding ones, and each asserts the presence of upstream contracts at start. A missing upstream file halts the notebook with a `FileNotFoundError` rather than silently falling back to defaults.

---

## Key decisions

A subset of the 25 documented decisions. Full decision log in `outputs/reports/decision_log_v1_2.txt`.

| ID | Category | Decision | Rationale |
|---|---|---|---|
| D001 | Data Quality | Random sampling (100,000 rows, seed 42) | Prevents temporal bias |
| D002 | Data Quality | Median imputation for numeric, mode for categorical | Robust to outliers |
| D005 | PD Calibration | 12-month observation window for Stage 1 | IFRS 9 requirement |
| D006 | PD Calibration | PIT PD (not TTC) | IFRS 9 requires current-conditions sensitivity |
| D007 | SICR Implementation | Proxy approach: top-decile of non-defaulted loans by `pd_estimate` | Dataset lacks current credit characteristics |
| D008 | Scenario Design | Four scenarios including Tail Risk | Comprehensive stress testing |
| D009 | Scenario Weights | 45 / 30 / 15 / 10 | Probability-weighted ECL per IFRS 9 §5.5.17 |
| D010 | Coverage Framing | Coverage diagnostic (ECL / total assets), not capital adequacy | CET1-to-total-assets comparison is a scale mismatch |
| D011 | Fairness Testing | Do not test risk drivers as protected classes; disclose geographic proxy results as UNRELIABLE | Risk drivers are not protected classes |
| D023 | EAD | CCF = 0 for term mortgages | Basel III CCF scope excludes term mortgages |
| D024 | Capital Framing | Portfolio-aggregate ECL for coverage, labelled diagnostic | Per-loan ECL vs. portfolio capital is a scale mismatch |
| D025 | Fairness | Do not conclude failure on geographic proxies | 4/5ths rule is misapplied to high-cardinality splits |

---

## GRC deliverables

| Artifact | Location | Description |
|---|---|---|
| Problem Definition Record (PDR) | `artifacts/PDR-PR004-v1.0.md` | Decision supported, population, target, constraints |
| Analytical Specification (AS) | `artifacts/AS-PR004-v1.0.md` | Research questions, hypotheses, variables, success criteria |
| Data Specification (DS) | `artifacts/DS-PR004-v1.0.md` | Data sources, lineage, missingness, leakage audit |
| Feature Decision Log (FDL) | `artifacts/FDL-PR004-v1.0.md` | Feature selection criteria, transformations, proxies |
| Methodology Document (MD) | `artifacts/MD-PR004-v1.0.md` | Candidates, selection, frozen parameters |
| Risk & Validation Plan (RVP) | `artifacts/RVP-PR004-v1.0.md` | Thresholds, stability, drift, fairness, decision rules |
| Decision Criteria Register (DCR) | `artifacts/DCR-PR004-v1.0.md` | Acceptable performance, drift, disparity, investigation triggers |
| Findings Register | `outputs/reports/findings_register.csv` | Per-finding severity, evidence, owner, remediation |
| Remediation Plan | `artifacts/Remediation-Plan-PR004-v1.0.md` | Sequenced actions with owners and deadlines |
| Approval Record | `artifacts/Approval-Record-PR004-v1.0.md` | Sign-off chain, monitoring conditions, next review |

Each artifact carries a version, freeze timestamp, and change log. Once frozen, an artifact can only be changed via a version bump with documented rationale.

---

## Technical stack

- **Language:** Python 3.10+
- **Core:** pandas, numpy, scipy, scikit-learn
- **Visualization:** matplotlib, seaborn
- **Governance artifacts:** JSON and CSV, each carrying `run_id` and `notebook_version`
- **Power BI exports:** fact and dimension tables, data dictionary, manifest

### Reproducibility

Every notebook run produces an audit log, a `run_id` on every artifact, and a SHA-256-16 hash on every saved file. The same random seed (42) is used throughout. The content hash of the final clean frame (`bc5263ca06d29037`) allows run-to-run comparison.

---

## Key documents

| Document | Description |
|---|---|
| `outputs/reports/ifrs9_compliance_report_v1_2.txt` | Compliance report with analyst placeholders |
| `outputs/reports/pd_lgd_ead_calibration_summary_v1_3.txt` | PD/LGD/EAD calibration results |
| `outputs/reports/decision_log_v1_2.txt` | 25 decisions with rationale |
| `outputs/reports/manifest_pipeline_v1_2.csv` | Every analysis artifact, with hash |
| `outputs/reports/notebook_03_parameters.json` | Calibration contract for downstream notebooks |
| `outputs/reports/notebook_04_results.json` | Stress-testing and fairness results |
| `powerbi_exports/data_dictionary.md` | Column-level documentation with limitations embedded |

---

## Related repositories

- [model-analysis-package](https://github.com/yourusername/model-analysis-package) — the MAP anchor: five governance deliverables, including this project's Stress Testing Framework
- [lending-club-model-validation](https://github.com/yourusername/lending-club-model-validation) — the companion independent validation

## License

MIT
