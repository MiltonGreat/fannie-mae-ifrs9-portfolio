# Fannie Mae IFRS 9 Compliance Framework

**A methodology demonstration of IFRS 9 Expected Credit Loss (ECL) on a mortgage portfolio.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![IFRS 9](https://img.shields.io/badge/IFRS%209-Methodology%20Demo-orange)
![OSFI](https://img.shields.io/badge/OSFI%20E--23-Applied-red)
![SR 11-7](https://img.shields.io/badge/SR%2011--7-Aligned-lightgrey)
![ECOA](https://img.shields.io/badge/ECOA-Not%20Assessed-red)

---

## Executive Summary

This project demonstrates an IFRS 9 Expected Credit Loss (ECL) framework applied to a Fannie Mae mortgage portfolio. It covers the full lifecycle from data preparation to regulatory reporting, with an emphasis on Governance, Risk, and Compliance (GRC).

**The work is a methodology demonstration, not a production implementation.** It shows how IFRS 9 concepts are implemented in code, how decisions are documented for audit, and where a real bank's production environment would need to substitute proprietary data and calibration.

### What this project demonstrates

- A working IFRS 9 ECL pipeline: PD/LGD/EAD calibration → three-stage classification → probability-weighted ECL across four scenarios
- A documented decision trail (12+ decisions with rationale and regulatory reference in the deliverable set; 20+ in the decision log)
- A data-quality discipline that surfaces leakage, missingness, and scope limitations as first-class findings rather than suppressing them
- A clean separation between what the code computes and what the analyst declares

### What this project does NOT demonstrate

- A production-calibrated ECL model — the parameters are assumption-based, not calibrated to a real bank's recovery data
- Independent validation — there is no second-line validation of this pipeline; the Validation Opinion is authored by the analyst, not by a validation notebook
- Fair-lending compliance — the Fannie Mae public dataset contains no protected-class attributes and no usable geographic proxies
- Canadian applicability — the pipeline is trained on U.S. data and is not validated for Canadian portfolios (OSFI E-23 scope restriction)
- IFRS 9 compliance — IFRS 9 compliance is a determination an institution makes against its regulatory obligations, not a claim a demonstration project can make for itself

The last two are the most important. A demonstration that over-claims is less useful than one that clearly names its limits.

---

## How to read this project

If you're new to IFRS 9 or to model governance, this section explains what the project is doing.

**What is IFRS 9?** IFRS 9 is the international accounting standard that governs how banks and other lenders recognize expected credit losses on their loans. Unlike the older "incurred loss" model — where a bank only booked a provision after a borrower actually defaulted — IFRS 9 requires the bank to estimate, on day one, how much it expects to lose over the loan's life, and to update that estimate every reporting period.

**Why this is hard.** The ECL is a forward-looking, probability-weighted estimate. It has three components:

- **PD (Probability of Default)** — the chance a borrower defaults over a given horizon (12 months for Stage 1 loans; lifetime for Stage 2 and 3)
- **LGD (Loss Given Default)** — the fraction of the exposure the bank expects to lose if default happens, after recoveries
- **EAD (Exposure at Default)** — the balance outstanding at the moment of default

`ECL = PD × LGD × EAD`, weighted across scenarios, discounted to today's value.

**What "three-stage classification" means.** IFRS 9 sorts loans into Stage 1 (performing, 12-month ECL), Stage 2 (significant increase in credit risk since origination, lifetime ECL), and Stage 3 (credit-impaired, lifetime ECL). The hard part is Stage 2: detecting *early* deterioration before the loan is actually in default.

**What this project demonstrates.** The full pipeline above, on public Fannie Mae data, with every methodological decision documented. What it does not do is calibrate PD, LGD, or EAD to a real bank's portfolio — those are assumption-based, and the project is transparent about which assumption each number rests on.

---

## Key Results

All figures are per loan unless labeled otherwise. Numeric values cite the pipeline's final run; where a value changed between pipeline versions, the note explains why.

| Metric | Value | Notes |
|---|---|---|
| Portfolio size | 100,000 loans | Random sample from Fannie Mae dataset (seed 42) |
| Default rate (observed) | 0.94–0.95% | Backward-looking frequency in the sample |
| PD (calibrated, unconditional) | 5.76% | Forward-looking estimate from the FICO/LTV/DTI/AGE risk score; clipped to floor 0.01%, cap 20% |
| Average LGD | 44.28% | Base 45% + age/LTV/property adjustments, clipped to [10%, 90%] — **inside the 40–60% benchmark band** |
| Average EAD (per loan) | ~$204,745 | After prepayment (8%) and curtailment (2%) adjustments. **EAD ratio 86.95%, above the 60–85% target band — flagged as AP-03** |
| Probability-weighted ECL (per loan) | ~$2,231 | Across four stress scenarios (45/30/15/10 weights) |
| Portfolio ECL (weighted) | ~$223M | Per-loan ECL × 100,000 |
| ECL / total assets (coverage diagnostic) | 22.3% | **Not a capital-adequacy finding** — see §"Capital coverage, correctly framed" |

> **Note on LGD.** An earlier version of this project reported average LGD of 14.18%. That figure came from a construction where LGD was a direct function of PD (clipped aggressively low). The revised pipeline uses the industry-standard base-LGD + adjustments construction (base 45%, plus age, LTV, and property-type adjustments, then clipped). The current figure of 44.28% sits inside the target band and is more defensible as an assumption-based estimate. This change is documented in the pipeline's changelog.

> **Note on EAD ratio.** The EAD ratio of 86.95% is above the 60–85% target band. This triggers investigation trigger **IT-03** and appears as Finding AP-03 in the register. It is not necessarily a methodology error — a portfolio weighted toward younger loans (before prepayment and curtailment have had time to reduce the balance) will naturally show a higher ratio. It is disclosed and flagged, not suppressed.

---

## Key Findings

### What worked well

- **IFRS 9 three-stage classification implemented**, with a documented proxy for SICR detection where the data lacks observed current credit characteristics.
- **Probability-weighted ECL** across four stress scenarios (Baseline, Adverse, Severely Adverse, Tail Risk), with the discount factor applied.
- **Decision log** documents each methodological choice with rationale, regulatory reference, and alternatives rejected.
- **Complete audit trail** — every notebook run is timestamped, hashed, and logged; every saved artifact carries `run_id` and `notebook_version`.
- **Data-quality discipline** — leakage, missingness, and scope limitations are surfaced as findings in the register, not suppressed as implementation details.

### What requires documented limitations

- **SICR detection is a proxy, not true SICR.** The dataset contains no observed current credit characteristics to compare against origination. Stage 2 is populated by the highest-risk decile of non-defaulted loans, which means Stage 2 ≈ 10% by construction. This is documented as Finding F-07 and Limitation L2.
- **Fair lending testing was not performed.** The Fannie Mae public dataset contains no protected-class attributes (race, ethnicity, sex, marital status, national origin, religion, age band) and no geographic fields usable as proxies (no state, region, census tract, or ZIP). This is a scope limitation, not a limitation of the model.
- **PD-LGD correlation is constructed, not calibrated from recovery data.** LGD is a function of PD through the collateral-segmentation adjustments; the resulting correlation is illustrative, not empirical. Documented as Limitation L3.
- **PD cap masks tail convexity.** The Tail Risk scenario's PD is capped at 20%, which mutes the convexity between the two most severe scenarios. The uncapped effect is reported in NB04 as sensitivity (PD-cap effect: ~30.7% understated). Documented as Limitation L6.
- **Capital coverage is a diagnostic, not an adequacy finding.** The CET1-to-total-assets comparison produces a scale-mismatched ratio that is not a valid capital test. See §"Capital coverage, correctly framed."

---

## Capital coverage, correctly framed

**This section is deliberately technical because the framing matters for regulatory credibility.**

An earlier version of this project reported a "capital impact ratio" of ~1,000% and characterized the result as "Inadequate under stress." That framing was wrong, and it has been corrected. Two things were happening:

**1. The denominator was the wrong capital.** CET1 is a ratio against risk-weighted assets (RWA), not against total assets. The pipeline assumed total assets of $1 billion and multiplied by a CET1 *ratio* (4.5%) to get an assumed capital figure of $45 million. That's not the CET1 capital of any real bank — it's a fabricated denominator that produces a ratio that looks alarming but isn't meaningful.

**2. The comparison was a scale mismatch.** Per-loan ECL × portfolio size gives portfolio-aggregate ECL (~$223M). Comparing that to $45M gives "995% covered" — which reads as "capital would be exhausted ten times over." But a real bank's CET1 capital is many multiples of $45M for a $223M stressed-ECL portfolio. The comparison is a numerical artifact.

**What the pipeline reports instead:**

| Metric | Value | What it means |
|---|---|---|
| Portfolio ECL as % of total assets (weighted) | 22.31% | A **coverage metric** — how much of the assumed asset base is consumed by expected losses |
| Portfolio ECL as % of total assets (tail) | 75.12% | Same, under the Tail Risk scenario |
| ECL / CET1 ratio | ~496% (weighted) | **Diagnostic only** — a scale-mismatched ratio, not a capital adequacy finding |

The pipeline labels the ECL/CET1 ratio as a diagnostic in the report itself, with the following disclaimer:

> "The ECL/CET1 ratio is a scale diagnostic, not a capital adequacy measure. A term mortgage portfolio does not hold capital equal to total assets. This ratio is provided for completeness only and should not be interpreted as a capital adequacy finding."

**What a real capital adequacy test would require:** risk-weighted assets (RWA) specific to the portfolio, actual CET1 capital of the institution, and the institution's own capital adequacy buffers (conservation buffer, stress buffer, any institution-specific add-ons). None of those are in this project.

This correction matters because getting the framing wrong on capital adequacy is the kind of error that gets a model risk finding cited against the institution. A project that says "Inadequate under stress" when its own math is a scale mismatch is less credible than one that says "Coverage diagnostic only; a real capital test requires RWA and institution-specific capital."

---

## Regulatory Compliance Status

Each framework is assessed against what the project **actually demonstrates**, not what it would require in production. The distinction matters: claiming compliance with a standard is a determination the institution makes against its regulatory obligations, not a claim a demonstration project can make for itself.

| Regulation | Status | Notes |
|---|---|---|
| **IFRS 9** | Methodology demonstration only | ECL measurement, three-stage classification, and probability weighting implemented. SICR uses a proxy (top-decile, not true SICR). Not suitable for provisioning without recalibration. |
| **OSFI E-23** | Aligned with limitations | Data quality and documentation practices followed. Fair-lending testing not performed (no protected-class attributes in dataset). Not validated for Canadian portfolios. |
| **SR 11-7** | Partially aligned | Model documentation and outcomes analysis implemented. **Independent validation was not performed** — the pipeline is a first-line methodology demonstration, not a second-line validation. Conceptual soundness is documented; effective challenge is not. |
| **ECOA** | Not assessed | Dataset lacks protected-class attributes. Cannot be tested on this data. |
| **Basel III** | Framework reference only | Capital framework concepts (CET1, RWA) are referenced but the pipeline does not implement a capital adequacy calculation. Coverage diagnostic only. |

A full regulatory mapping — including the specific requirements addressed, the evidence, and the gaps — is in `outputs/reports/regulatory_mapping_v2.csv` and `outputs/reports/compliance_report_v2.txt`.

> **Read with:** the Validation Opinion document, which states the analyst's final determination on each framework's applicability. The README reports what the notebooks demonstrate; the Validation Opinion states what the analyst concludes.

---

## Project Objectives

- Demonstrate IFRS 9 ECL methodology using publicly available mortgage data.
- Build an end-to-end pipeline from raw data to regulatory reporting.
- Document every methodological decision with rationale and audit trail.
- Assess **coverage** under stress scenarios (not capital adequacy).
- Implement fair lending testing *where protected attributes are available* — acknowledging that the Fannie Mae public dataset is not one of those datasets.
- Generate audit-ready artifacts suitable for internal review.

---

## Notebooks Overview

| Notebook | Focus | Key Outputs | Regulatory Link |
|---|---|---|---|
| 01 Data Loading | Quality & cleaning | Cleaned dataset (100,000 × 145), missing indicators, assumptions register, `data_quality_summary.json`, findings register (8 entries) | OSFI E-23 |
| 02 EDA | Risk factor analysis | Risk drivers, correlations, vintage patterns, t-test results, segment analyses | SR 11-7 (outcomes analysis) |
| 03 Calibration | PD/LGD/EAD calibration | IFRS 9 parameters, stage classification, LGD/EAD construction, ECL per stage and per scenario, `notebook_03_parameters.json` | IFRS 9 |
| 04 Stress Testing | Scenarios & coverage | Scenario ECLs, PD-cap sensitivity, severity check, coverage diagnostic, fairness assessment (UNRELIABLE at high cardinality) | IFRS 9, ECOA |
| 05 Reporting | Dashboard & report | Compliance report (with `[ANALYST: …]` placeholders), decision log, manifest | All |
| 06 Power BI Export | Data export | Fact and dimension tables, data dictionary, manifest | Reporting |

**Run the notebooks in order.** Each depends on artifacts produced by the preceding ones, and each asserts the presence of upstream contracts at start. A missing upstream file halts the notebook with a `FileNotFoundError` rather than silently falling back to defaults.

---

## IFRS 9 Parameters

All figures are per loan unless labeled otherwise.

| Parameter | Value | Description |
|---|---|---|
| Default rate (observed) | 0.94–0.95% | Backward-looking frequency in sample |
| PD (unconditional) | 5.76% | Forward-looking estimate; floor 0.01%, cap 20% |
| Average LGD | 44.28% | Base 45% + collateral adjustments; floor 10%, cap 90% |
| Average EAD | ~$204,745 | After prepayment (8%) and curtailment (2%) adjustments |
| EAD ratio (EAD / ORG_UPB) | 86.95% | Above 60–85% target band; flagged as AP-03 |
| ECL (unstressed, per loan) | ~$835 | PD × LGD × EAD × discount factor |
| Probability-weighted ECL (per loan) | ~$2,231 | Across four scenarios with weights |
| Discount factor | 1 / (1 + 0.035) | EIR = 3.5% (industry average) |

### IFRS 9 stage distribution

| Stage | Share | ECL Basis |
|---|---|---|
| Stage 1 | ~89.1% | 12-month ECL |
| Stage 2 | ~9.9% | Lifetime ECL (proxy SICR) |
| Stage 3 | ~1.0% | Lifetime ECL (credit-impaired) |

Stage 2 population is populated using a proxy — the highest-risk decile of non-defaulted loans by `pd_estimate`. The mechanism is explicitly disclosed in the notebook's output, not hidden. Since the decile is computed on the non-Stage-3 population, Stage 2 is approximately 10% by construction, not by observed credit deterioration. This is documented as Finding F-07.

---

## Scenario Analysis

| Scenario | PD | LGD | ECL (per loan) | Weight | Weighted ECL |
|---|---|---|---|---|---|
| Baseline | 5.76% | 44.28% | ~$835 | 45% | ~$376 |
| Adverse | 10.37% | 53.13% | ~$1,803 | 30% | ~$541 |
| Severely Adverse | 17.28% | 66.41% | ~$3,756 | 15% | ~$563 |
| Tail Risk | 20.00% (capped) | 88.55% | ~$7,512 | 10% | ~$751 |

**Probability-weighted ECL (per loan): ~$2,231.**

### PD cap sensitivity

The Tail Risk scenario's PD is capped at 20% (the config's `pd_cap`). Without the cap, the pre-cap PD would be ~26.1%, and the uncapped ECL would be ~$16,858 per loan, versus $12,901 with the cap. The cap **understates Tail Risk ECL by ~30.7%**.

This is a deliberate stability measure, not a calibration error. It is documented as Limitation L6, and the uncapped value is reported alongside the capped value in NB04 so the effect is disclosed rather than hidden.

### Severity calibration check

The Severely Adverse ECL as a rate of EAD is compared against the 2008–2010 historical average (1.44%). The pipeline reports a ratio of ~1.27× historical — below the 1.5× "severely adverse" threshold in the Register.

This triggers investigation trigger **IT-05**. It is a directional check, not a reproduction of the 2008 crisis, and the notebook labels it as such. The comparison answers the question *"Is the scenario severe enough?"* It does not answer the question *"Would the portfolio have survived the 2008 crisis?"* — that would require the historical defaults themselves, not an average rate.

---

## Fairness Testing

| Assessment | Status |
|---|---|
| Fair lending testing | **Not performed** |
| Reason | Dataset contains no protected-class attributes or usable geographic proxies |
| Attributes checked | race, ethnicity, sex, gender, marital status, national origin, religion, age band, state, region, census tract, ZIP, MSA |
| Attributes found | None usable as protected-class proxies |

### What this means

Fair lending testing under ECOA (US), OSFI E-23 s.6.0 (Canada), and equivalent regimes requires testing on protected classes. The Fannie Mae public dataset does not contain them, and fair lending compliance **cannot be verified from this data**.

The pipeline **does** run fairness testing on the geographic proxies it has — state, MSA, ZIP — and reports the results with an explicit `UNRELIABLE` flag, because the 4/5ths rule at cardinality 52 / 328 / 638 produces degenerate results (many groups with zero defaults → DI = 0.00%, which is not evidence of discrimination). This is documented as Finding F-05 (Fairness testing incomplete — MEDIUM, documented limitation) and Limitation F-L2.

### What the pipeline does not do

The pipeline does **not** test disparate impact on credit risk drivers (FICO, property type, occupancy status). Those are legitimate underwriting variables, not protected classes, and testing them under a "fairness" label would produce false positives — differences that are expected and legal.

### What a production fair lending assessment would require

Data containing protected attributes or geographic identifiers (ZIP code, census tract, ideally census-tract-level demographics). For US mortgage portfolios, this is available through HMDA reporting. For Canadian portfolios, an equivalent disclosure regime would need to be identified.

---

## GRC Deliverables

### Governance artifacts

| Artifact | Location | Description |
|---|---|---|
| Problem Definition Record (PDR) | `artifacts/PDR-PR004-v1.0.md` | Decision supported, population, target, constraints, stakeholders |
| Analytical Specification (AS) | `artifacts/AS-PR004-v1.0.md` | Research questions, hypotheses, variables, assumptions, success criteria |
| Data Specification (DS) | `artifacts/DS-PR004-v1.0.md` | Data sources, lineage, inclusion/exclusion, missingness, leakage audit |
| Feature Decision Log (FDL) | `artifacts/FDL-PR004-v1.0.md` | Feature selection criteria, transformations, proxies |
| Methodology Document (MD) | `artifacts/MD-PR004-v1.0.md` | Candidates, selection, frozen parameters, performance measures |
| Risk & Validation Plan (RVP) | `artifacts/RVP-PR004-v1.0.md` | Thresholds, stability, drift, fairness, decision rules, escalation |
| Decision Criteria Register (DCR) | `artifacts/DCR-PR004-v1.0.md` | Acceptable performance, drift, disparity, investigation triggers, failed-validation tiers, freeze record |
| Findings Register | `outputs/reports/findings_register.csv` + `artifacts/Findings-PR004-v1.0.md` | Per-finding severity, evidence, owner, remediation, status |
| Validation Opinion | `artifacts/Validation-Opinion-PR004-v1.0.md` | Analyst's determination — states Approved / Approved with conditions / Rejected, per the frozen Register |
| Remediation Plan | `artifacts/Remediation-Plan-PR004-v1.0.md` | Sequenced actions with owners, deadlines, success criteria |
| Approval Record | `artifacts/Approval-Record-PR004-v1.0.md` | Sign-off chain, monitoring conditions, next review date |

Each artifact carries a version, freeze timestamp, and change log. Once frozen, an artifact can only be changed via a version bump with documented rationale.

### Reports

| Report | Location | Description |
|---|---|---|
| Compliance report | `outputs/reports/ifrs9_compliance_report_v1_2.txt` | Structured report with executive summary **placeholders** (`[ANALYST: …]`), compliance table, key results (numbers only), limitations list, sign-off block |
| Calibration summary | `outputs/reports/pd_lgd_ead_calibration_summary_v1_3.txt` | PD/LGD/EAD calibration results |
| Assumptions narrative | `outputs/reports/assumptions_register_narrative.md` | Human-readable assumptions register |
| Decision log | `outputs/reports/decision_log_v1_2.txt` | 25 decisions with rationale, regulatory reference |
| Pipeline manifest | `outputs/reports/manifest_pipeline_v1_2.csv` | Every analysis artifact, with SHA-256 hash |

### Data contracts

| Artifact | Location | Description |
|---|---|---|
| `notebook_03_parameters.json` | `outputs/reports/notebook_03_parameters.json` | All config values and computed parameters from NB03 — the contract NB04 and NB05 read |
| `notebook_04_results.json` | `outputs/reports/notebook_04_results.json` | All scenario results, coverage metrics, severity check, fairness results from NB04 |
| `data_quality_summary.json` | `outputs/reports/data_quality_summary.json` | Machine-readable data quality assessment from NB01 |

Each contract carries `run_id` and `notebook_version`. Downstream notebooks assert these before proceeding.

### Power BI exports

| Export | Location | Description |
|---|---|---|
| Fact tables | `powerbi_exports/fact_*.csv` | Loan portfolio, segment analysis, scenario analysis, capital coverage, IFRS 9 stages, fairness testing |
| Dimension table | `powerbi_exports/dim_segment.csv` | Segment dimension (FICO, LTV, property type, age, DTI) |
| Summary metrics | `powerbi_exports/summary_metrics.csv` | Key metrics for dashboard display |
| Data dictionary | `powerbi_exports/data_dictionary.md` | Column-level documentation with limitations embedded |

Exports are promoted from a per-run staging directory to the final `powerbi_exports/` location, with an assertion that the promoted set is exactly the spec'd set. Stale artifacts from prior runs are removed with a logged `WARNING`.

---

## Audit Trail

Every notebook writes a timestamped log to `logs/`. Each run has a unique `RUN_ID` (timestamp-based), and the log records every significant decision with its time of execution.

Artifacts saved by every notebook carry:
- `run_id` (timestamp-based)
- `notebook_version` (semantic version, e.g., `1.2`, `1.3`)

Every saved file's SHA-256-16 hash is recorded in the audit log and the manifest.

The **prompt library** used for the AI-assisted notebooks — including the P1–P9 prompt structure, boundary enforcement prompts, ambiguity flagging, and edit-type taxonomy — is documented separately. The pipeline was developed with AI assistance under the discipline described in the prompt library; the analyst owned every analytical decision and every interpretation.

---

## Key Decisions

A subset of the 25 documented decisions. Full decision log in `outputs/reports/decision_log_v1_2.txt`.

| ID | Category | Decision | Rationale |
|---|---|---|---|
| D001 | Data Quality | Random sampling (100,000 rows, seed 42) | Prevents temporal bias from chronological ordering |
| D002 | Data Quality | Median imputation for numeric, mode for categorical | Robust to outliers; preserves distribution |
| D003 | Feature Engineering | LTV threshold: 80% | Aligns with PMI requirement |
| D004 | Feature Engineering | FICO bands: 680 / 720 / 760 | Industry standard for risk segmentation |
| D005 | PD Calibration | 12-month observation window for Stage 1 | IFRS 9 requirement |
| D006 | PD Calibration | PIT PD (not TTC) | IFRS 9 requires current-conditions sensitivity |
| D007 | SICR Implementation | Proxy approach: top-decile of non-defaulted loans by `pd_estimate` | Dataset lacks current credit characteristics required for true SICR |
| D008 | Scenario Design | Four scenarios including Tail Risk | Comprehensive stress testing with disclosed severity check |
| D009 | Scenario Weights | 45 / 30 / 15 / 10 | Probability-weighted ECL per IFRS 9 §5.5.17 |
| D010 | Coverage Framing | Coverage diagnostic (ECL / total assets), not capital adequacy | CET1-to-total-assets comparison is a scale mismatch |
| D011 | Fairness Testing | Do not test risk drivers as protected classes; disclose geographic proxy results as UNRELIABLE | Risk drivers are not protected classes; geographic proxies at high cardinality produce degenerate results |
| D023 | EAD | CCF = 0 for term mortgages | Basel III CCF scope excludes term mortgages (no undrawn commitment) |
| D024 | Capital Framing | Portfolio-aggregate ECL for coverage, labelled diagnostic | Per-loan ECL vs. portfolio capital is a scale mismatch |
| D025 | Fairness | Do not conclude failure on geographic proxies | 4/5ths rule is misapplied to high-cardinality splits |

---

## Getting Started

### Prerequisites

- Python 3.10+
- Jupyter Notebook or JupyterLab
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/fannie-mae-ifrs9-portfolio.git
cd fannie-mae-ifrs9-portfolio

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
