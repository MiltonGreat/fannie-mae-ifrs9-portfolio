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
- **Independent validation** — there is no second-line validation of this pipeline; **no Validation Opinion exists for this project, by design** (see "The framing correction" and the "GRC deliverables" section below)
- **Fair-lending compliance** — the Fannie Mae public dataset contains no protected-class attributes and no usable geographic proxies
- **Canadian applicability** — the pipeline is trained on U.S. data and is not validated for Canadian portfolios (OSFI E-23 scope restriction)
- **IFRS 9 compliance** — IFRS 9 compliance is a determination an institution makes against its regulatory obligations, not a claim a demonstration project can make for itself

**The last two are the most important.** A demonstration that over-claims is less useful than one that clearly names its limits.

**On the absence of a Validation Opinion.** The Lending Club project (the companion repo) has a Validation Report because it is a *validated model*. The Fannie Mae project is a *framework* — a methodology demonstration, not a production model. **No Validation Opinion exists for the Fannie Mae project, and that is deliberate.** A framework that has not been validated should not have a Validation Opinion claiming it has. The framework's deliverables document what the pipeline computes and what it does not; the Validation Opinion is the Lending Club project's artifact because the Lending Club project is the one that was independently validated.

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

> **Note on the severity check.** The Severely Adverse scenario's ECL rate is 1.27× the 2008–2010 historical average, which is **below** the 1.5× target in the Decision Criteria Register. The notebook logs an AP-05 warning and an IT-05 investigation trigger. The correct reading is: the assumed scenario multipliers produce a scenario the register considers *insufficiently severe*, not that the portfolio is safe. The 3.0× PD multiplier is an assumption with a precedent label ("2008 financial crisis"), not a calibrated parameter. A production implementation would derive the multiplier from a macro-to-PD linkage; this notebook does not implement that linkage. This is documented as Limitation L9.

> **Note on the PD cap.** The PD cap does not bind in this run — Tail Risk PD (4.29%) is below the 20% cap. The cap-sensitivity analysis therefore reports +0.0% understatement. The mechanism is in place for runs where base PD is higher; in this run it is a null result, not a finding.

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

**What a true SICR assessment would require.** IFRS 9 §5.5.9 requires assessing whether credit risk has *increased significantly since initial recognition*, using all reasonable and supportable information available without undue cost or effort. The standard's indicators include changes in internal credit ratings or scores since origination, and significant increases in the credit risk of other instruments of the same borrower. This dataset contains origination characteristics and a default outcome; it does not contain current credit scores, current delinquency status, or any observation of the borrower's circumstances at the reporting date. A true SICR assessment cannot be made with origination-only data. The proxy is a cross-sectional risk ranking, not a change-since-origination assessment. The notebook documents this, and the honest reading of the Stage 2 share is "top-decile proxy," not "SICR."

**What a true 30-DPD backstop would require.** IFRS 9 §5.5.11 creates a *rebuttable* presumption that 30 days past due is a SICR trigger — not an automatic rule. The notebook applies the backstop without rebuttal, which is conservative (it may over-assign Stage 2) but is a simplification, not a standard-compliant implementation. The notebook documents this distinction.

---

## Scenario analysis

| Scenario | PD | LGD | ECL (per loan) | Weight | Weighted ECL |
|---|---|---|---|---|---|
| Baseline | 0.95% | 44.28% | ~$835 | 45% | ~$376 |
| Adverse | 1.72% | 53.13% | ~$1,803 | 30% | ~$541 |
| Severely Adverse | 2.86% | 66.41% | ~$3,756 | 15% | ~$563 |
| Tail Risk | 4.29% (capped) | 88.55% | ~$7,512 | 10% | ~$751 |

**Probability-weighted ECL (per loan): ~$2,231.**

### What the scenario design does and does not do

The scenario design has two layers that are not connected:

- **Layer 1 — macro narratives** (GDP, unemployment, HPI). Documented for context, **not used in the calculation**.
- **Layer 2 — PD/LGD multipliers.** Direct inputs to the ECL arithmetic.

IFRS 9 §5.5.18 requires incorporating forecasts of future economic conditions. In this notebook, the economic conditions are narrated but not modelled. A production implementation would map the macro path to PD/LGD (via a satellite model or a historical regression), so a change in the unemployment assumption changes the ECL. Here, changing the macro dict changes nothing. **This is a methodology demonstration of the arithmetic of probability-weighted ECL, not an implementation of the economic linkage that §5.5.18 requires.**

### PD cap sensitivity

The Tail Risk scenario's PD is capped at 20% (the config's `pd_cap`). In this run, the cap **does not bind** — Tail Risk PD (4.29%) is well below the cap, so the cap-sensitivity effect is +0.0%. The mechanism is documented as Limitation L6, and the uncapped value is reported alongside the capped value in NB04 so the effect is disclosed rather than hidden. A run with a higher base PD would show a non-zero effect.

### Severity calibration check

The Severely Adverse ECL as a rate of EAD is compared against the 2008–2010 historical average (1.44%). The pipeline reports a ratio of **1.27×** historical — **below** the 1.5× "severely adverse" threshold in the Register.

This triggers investigation trigger **IT-05**. It is a directional check, not a reproduction of the 2008 crisis, and the notebook labels it as such. The comparison answers the question *"Is the scenario severe enough?"* It does not answer *"Would the portfolio have survived the 2008 crisis?"* — that would require the historical defaults themselves, not an average rate. And the comparison is **circular in a specific way** the disclaimer names: the Severely Adverse ECL is produced by the 3.0× PD multiplier, so the check tests whether the multiplier produces a result above the target, not whether the multiplier is right.

### The 3.0× multiplier

The Severely Adverse PD multiplier (3.0×) and LGD multiplier (1.5×) are **assumptions with a precedent label** — "the 2008 financial crisis" — not calibrated parameters. A validator should ask: what evidence supports 3.0× rather than 2.0× or 4.0×? The notebook's answer is the precedent, not a derivation. The notebook does not validate the multipliers; it tests them against a target and reports a miss (1.27× < 1.5×).

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

**What "UNRELIABLE" means, and why it is not "Non-Compliant."** The 4/5ths rule is a disparate-impact test designed for binary or low-cardinality protected attributes (race, ethnicity, sex, marital status). Applied to geographic splits with hundreds of groups, the test is being used outside its intended scope. The `UNRELIABLE` flag is the notebook's mechanism for saying: *this test is degenerate; do not read the DI ratio as a finding*. The distinction matters: `UNRELIABLE` means *the test cannot tell you*, while `Non-Compliant` would mean *the test was performed and failed*. The notebook never writes the latter. The analyst, writing the Validation Opinion, can honestly write "Not Assessed" rather than "Non-Compliant" — because the notebook's flag is the evidence for that status.

**Proxy risk that is not tested.** A variable correlated with default may be a proxy for something else — `NUM_BOR` (number of borrowers), for example, is plausibly a proxy for household structure and income pooling rather than a causal risk driver. A model that uses `NUM_BOR` may pick up a protected-class-correlated signal even when the variable itself is facially neutral. NB02 surfaces this question; the pipeline does not resolve it. **This is the deeper fair-lending exposure the pipeline does not test**, and the honest framing is that the pipeline names it rather than claiming to have addressed it.

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

### Reading the regulatory status column

The five statuses above are **analyst-level determinations of the project's scope**, not notebook-level verdicts. The distinction matters:

- **"Methodology demonstration only"** — the pipeline implements the arithmetic of the standard, but the data does not support the full requirement.
- **"Aligned with limitations"** — the practices follow the regulation's intent, but at least one required element is absent (in this case, fair-lending testing).
- **"Partially aligned"** — some requirements are implemented, others are not; the gaps are named.
- **"Not assessed"** — the data does not support the test. This is materially different from "failed the test." It means the test could not be performed.
- **"Framework reference only"** — concepts are invoked but the implementation is not a calculation of the framework.

**"Aligned with limitations" vs. "Compliant with limitations."** The README deliberately uses "aligned" rather than "compliant." "Compliant with limitations" is a status an institution writes in a Validation Opinion, against its own regulatory obligations. A demonstration project cannot award itself that status — the notebook forbids the word "Compliant" in its output (see the claim-authorization section). What the project *can* say is that its practices are aligned with the regulation's intent, with named limitations.

### Citation verification

Every regulatory citation in this README is intended to be verified against the standard's actual text. Two citation corrections carried through the pipeline:

- **OSFI B-13 s.3.2 → OSFI E-23 Principle 3.2 (traceability).** B-13 is the Technology and Cyber Risk Management guideline, not the model risk guideline. The random seed's reproducibility purpose is served by E-23's traceability property, which is the correct citation.
- **OSFI E-23 s.4.3 → OSFI E-23 s.4.2.** The data quality language in E-23 sits in section 4.2 ("Data"), not 4.3. The notebook and decision log cite 4.2.

The remaining citations — IFRS 9 §5.5.3, §5.5.9, §5.5.11, §5.5.17, §5.5.18 — should be verified against the IFRS 9 PDF before treating them as final. The notebook structure is designed to make any correction a one-line change.

---

## The claim-authorization discipline

**This is the feature that most distinguishes the project.** Every notebook in this pipeline deliberately stops short of writing a compliance verdict. The claim-authorization discipline is enforced in code:

- **NB03** declares its scope in code comments and prints: *"IFRS 9 methodology demonstration; SICR proxied, not satisfied."* The word "Compliant" is not used.
- **NB04** asserts that no fairness result contains a `PASS` token, and that high-cardinality results are flagged `UNRELIABLE`.
- **NB05** asserts that the executive summary is a placeholder and that the report contains no `Compliant`, `Non-Compliant`, `PASS`, `FAIL`, `Adequate`, `Inadequate`, or `Validated` tokens. The self-check message announces "completed," not "PASS."
- **NB05** verifies that every artifact cited in the compliance table exists before the report is written (so the evidence column is checkable, not a dead reference).
- **NB06** asserts that no ROC/AUC function is in the namespace (C5 scope guard — AUC does not apply to a calibration model).

**Why this matters.** The pipeline produces **evidence**, not **verdicts**. Every compliance status in the report is an `[ANALYST: insert status]` placeholder. The analyst writes the verdict; the notebook supplies the evidence. This is deliberate — it keeps the tool from over-claiming and keeps the human in the loop.

**The practical consequence.** A reader of this README who asks "is this IFRS 9 compliant?" will not find an answer here, because the project does not claim one. The answer belongs to the analyst, in the Validation Opinion — which is not in this repo, because this project is a framework, not a validated model.

**What a governance artifact that reports its own limitations accurately looks like.** The final summary in NB05 says: *"This summary reports what the analysis actually produced, including limitations. A governance artifact that reports its own limitations accurately is more valuable than one that overstates its capabilities."* The v1.3 revision demonstrates this sentence in both directions. Three defects from v1.2 — a dead artifact reference, a success message that contained the token it forbade, and two stale regulatory citations — are fixed. A fourth issue surfaced by the run itself: the reconciliation check between the dataframe's `pd_estimate` mean and the NB03 contract value fires a warning that is definitional (per-loan PD vs. portfolio default rate), not a data defect. The warning should be reclassified, and the drift should be added to the Findings Register. The artifact reports the drift; it does not yet resolve it. That is the next version's work, and it is the discipline the sentence describes.

---

## Notebooks overview

| Notebook | Focus | Key Outputs | Regulatory Link |
|---|---|---|---|
| 01 Data Loading | Quality & cleaning | Cleaned dataset (100,000 × 175), missing indicators, assumptions register, `data_quality_summary.json` | OSFI E-23 |
| 02 EDA | Risk factor analysis | Risk drivers, correlations, vintage patterns, t-test results | SR 11-7 (outcomes analysis) |
| 03 Calibration | PD/LGD/EAD calibration | IFRS 9 parameters, stage classification, LGD/EAD construction, ECL per stage and per scenario, `notebook_03_parameters.json` | IFRS 9 |
| 04 Stress Testing | Scenarios & coverage | Scenario ECLs, PD-cap sensitivity, severity check, coverage diagnostic, fairness assessment (UNRELIABLE at high cardinality) | IFRS 9, ECOA |
| 05 Reporting | Dashboard & report | Compliance report (with `[ANALYST: …]` placeholders), decision log, manifest | All |
| 06 Power BI Export | Data export | Fact and dimension tables, data dictionary, manifest | Reporting |

**Run the notebooks in order.** Each depends on artifacts produced by the preceding ones, and each asserts the presence of upstream contracts at start. A missing upstream file halts the notebook with a `FileNotFoundError` rather than silently falling back to defaults.

### What each notebook teaches

- **NB01** — how a central assumptions register works as a single source of truth; how audit logging turns "reproducibility" from a word into a timestamped artifact; how missing indicators are created *before* imputation, preserving the information that a value was absent.
- **NB02** — how segment analysis works (banding a continuous variable, computing default rates per band, testing with chi-square); how correlation analysis works and what it does not tell you (the exclusion of `delinquency_state` as a target transform; the FICO/LTV multicollinearity finding); how vintage analysis works and why a three-year window with a confounded age effect is a *pattern*, not a *finding*.
- **NB03** — how PD calibration works (unconditional PD, lifetime PD via survival, clipping); how LGD calibration works (base + linear adjustments); how EAD calculation works (prepayment, curtailment, and the CCF=0 judgment for term mortgages); how SICR staging works *and what happens when you cannot actually do it* (proxy, disclosed).
- **NB04** — how scenario design works (four scenarios with documented macro rationales — narrated, not modelled); how probability-weighted ECL works (sum of scenario ECL × weight); how capital coverage differs from capital adequacy; how fair-lending testing works when it cannot be performed (the UNRELIABLE flag as the precondition for "Not Assessed").
- **NB05** — how a compliance report separates what the tool computes from what the analyst declares; how the compliance matrix stays honest (every row has a verified evidence artifact and a status cell the tool is forbidden from filling); how a decision log makes every choice traceable to a regulatory provision; why "Compliant with limitations" belongs in the Validation Opinion, not in the tool.
- **NB06** — how a star-schema export uses the vocabulary of dimensions and facts without encoding the joins (the schema lives in Power BI's model view, not the CSVs); how contract-read + reconciliation assertion replaces recomputation (the pattern that keeps the export consistent with the analysis); how the spec is itself a decision (omitting `dim_date` because no fact has a date key is a decision recorded in the spec, not a detection performed by code); how the data dictionary is the traceability artifact for the dashboard audience who never sees the notebooks.

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

**Ten** project-governance artifacts support this framework. They live in the `governance/` folder. Each carries a version, freeze timestamp, and change log; once frozen, an artifact can only be changed via a version bump with documented rationale.

| Artifact | File | Description |
|---|---|---|
| Problem Definition Record | `governance/problem-definition.md` | Decision supported, population, target, constraints |
| Analytical Specification | `governance/analytical-specification.md` | Research questions, hypotheses, variables, success criteria |
| Data Specification | `governance/data-specification.md` | Data sources, lineage, missingness, leakage audit, fitness-for-purpose determination |
| Feature Decision Logs | `governance/feature-decision-logs.md` | Feature selection criteria, transformations, proxies |
| Methodology Document | `governance/methodology-document.md` | Candidates, selection, frozen parameters, mechanism for each limitation |
| Risk & Validation Plan | `governance/risk-validation-plan.md` | Thresholds, stability, drift, fairness, decision rules, self-assessment components |
| Decision Criteria Register | `governance/decision-criteria-register.md` | Acceptable performance, drift, disparity, investigation triggers, guard results |
| Consolidated Findings Register | `governance/consolidated-findings-register.md` | Cross-reference of findings across the pipeline (the markdown consolidation of the FM findings register) |
| Approval Record | `governance/approval-record.md` | Sign-off chain, monitoring conditions, next review |
| Analysis Code & Results | `governance/code-analysis-results.md` | Index to the notebooks and a summary of their results |

### One governance artifact that is not in the folder, and why

- **Validation Opinion.** The Lending Club project has a Validation Opinion (in progress). The Fannie Mae project does not — **by design**. The Fannie Mae project is a *framework*, not a validated model. A framework that has not been independently validated should not have a Validation Opinion claiming it has. This is the deliberate design decision described in "What this framework does NOT demonstrate."

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
| `outputs/reports/pd_lgd_ead_calibration_summary_v1_2.txt` | PD/LGD/EAD calibration results |
| `outputs/reports/decision_log_v1_2.txt` | 25 decisions with rationale |
| `outputs/reports/manifest_pipeline_v1_2.csv` | Every analysis artifact, with hash |
| `outputs/reports/notebook_03_parameters.json` | Calibration contract for downstream notebooks |
| `outputs/reports/notebook_04_results.json` | Stress-testing and fairness results |
| `powerbi_exports/data_dictionary.md` | Column-level documentation with limitations embedded |
| `governance/` | Ten project-governance artifacts — see the "GRC deliverables" section above |

**On the two registers.** The `governance/consolidated-findings-register.md` is the **FM Consolidated Findings Register** — the 13 methodology limitations (L1–L10 plus the three NB01 data-preparation findings). The `outputs/reports/findings_register.csv` is **NB01's data-preparation findings register** — the three findings NB01 produced in its Phase 8. **The two files are different:** the consolidated register is the full cross-pipeline register; the NB01 register is one notebook's contribution to it. A reader who wants the complete picture reads the consolidated register; a reader who wants to understand NB01's specific findings reads the NB01 register.

---

## Related repositories

- [model-analysis-package](https://github.com/MiltonGreat/model-analysis-package) — the MAP anchor: the governance framework (AI Governance Policy, Regulatory Gap Analysis, Vendor Governance Review Template) and the package deliverables
- [lending-club-model-validation](https://github.com/MiltonGreat/credit-risk-model-validation) — the companion independent validation

## License

MIT
