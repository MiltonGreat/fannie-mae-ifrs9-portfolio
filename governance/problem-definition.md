# Problem Definition Record (A01)

| Metadata Field | Value |
| :--- | :--- |
| **Document ID** | `FM-PDR-001` |
| **Version** | `0.2.0-draft` (Forward-Looking Planning Draft) |
| **Run ID / Timestamp** | `YYYYMMDD_HHMMSS` *(link to active execution run log)* |
| **Author** | *(Specify Author / Role)* |
| **Project Scope** | IFRS 9 Expected Credit Loss (ECL) Methodology Demonstration (Public Data) |
| **Regulatory Alignment References** | OSFI E-23, SR 26-2, ECOA *(illustrative/defensibility alignment; not a compliance certification)* |
| **Cross-Referenced Artifacts** | `FM-DCR-001`, `FM-CFR-001`, `FM-AR-001`, `A03` (Data Specification), `A14` (Findings), `A16` (Remediation Plan) |

> **Governance & Scope Notice:** This document is written from a forward-looking planning perspective to define intended design, governance purpose, and auditability for the ongoing project, paired with planned/recorded execution parameters where applicable. This project is a **methodology demonstration on public data**, not a production allowance and not an independent validation opinion.

---

## 1. What business problem does the model address?

* **Intended Design & Governance Purpose:** Define a transparent, reproducible IFRS 9 Expected Credit Loss (ECL) demonstration framework ($\text{ECL} = \text{PD} \times \text{LGD} \times \text{EAD}$ probability-weighted across scenarios, with stage allocation) that shows how allowance calculations, stress/scenario mechanics, and audit trails are structured to support defensibility against OSFI E-23, SR 26-2, and ECOA principles.
* **Planned / Applied Execution Context:** A small bank must recognize an allowance for expected credit losses on its mortgage portfolio and demonstrate to its regulator that the allowance is defensible. Under IFRS 9, this requires computing Expected Credit Loss ($\text{ECL}$) as $\text{PD} \times \text{LGD} \times \text{EAD}$, probability-weighted across scenarios, with loans classified into stages. The project produces that ECL figure on public Fannie Mae data, with a documented audit trail sufficient for internal and regulatory review. It is a demonstration of the methodology, not a production allowance, and not an independent validation opinion.

---

## 2. What decisions does the model support?

* **Intended Design & Governance Purpose:** Explicitly map planned analytical outputs to governance decisions, decision-makers, and review cadences so decision authority remains clearly bounded.
* **Planned / Applied Execution Context:**

| # | Decision | Decision-Maker | Frequency |
| :--- | :--- | :--- | :--- |
| **D1** | ECL allowance to recognize this period | CFO / Controller | Quarterly |
| **D2** | Whether the allowance is adequate under stress | CRO / Treasurer | Quarterly |
| **D3** | Whether CET1 capital is adequate under stress | CRO / Treasurer | Quarterly — separate from D1 |
| **D4** | Whether to adjust underwriting standards | Chief Credit Officer | Annual |
| **D5** | Whether the model is fit for continued use | Model Risk Committee | Annual |

* **What the pipeline supports (demonstration scope):**
  * A methodology demonstration for a portfolio, technical, or regulatory audience.
  * Pipeline outputs (`PD`, `LGD`, `EAD`, staging, probability-weighted `ECL`) as evidence that the arithmetic works.

---

## 3. What decisions does the model not support?

* **Intended Design & Governance Purpose:** Establish hard non-use and decision-authority boundaries prior to execution so demonstration outputs are not misapplied to operational, capital, or compliance decisions.
* **Planned / Applied Execution Context:**
  * The model does **not** support: individual loan pricing or underwriting; regulatory capital under Basel III IRB; compensation or performance evaluation; fair lending compliance certification; any decision affecting a specific borrower; or Canadian portfolio decisions without recalibration.
  * The pipeline does **not** support: a production ECL estimate, a provisioning decision, a capital adequacy determination, or a regulatory compliance claim.
  * The pipeline does **not** decide whether the framework is approved, whether the methodology is production-ready, whether the ECL estimate is correct, or whether the framework is IFRS 9-compliant. Those judgments belong to the analyst in the Approval Record (`FM-AR-001`) and any independent validation opinion.

---

## 4. What is the population and unit of analysis?

* **Intended Design & Governance Purpose:** Specify the target source population, sampling design, unit of analysis assumptions, and verification dependencies prior to execution.
* **Planned / Applied Population Parameters:**

| Field | Value |
| :--- | :--- |
| **Source population** | Fannie Mae Single-Family Loan Performance (public tape; `train4.csv` from `Fannie Mae.zip`) |
| **Full population size** | `1,672,822` rows |
| **Sample size** | `100,000` rows |
| **Sample method** | Random sample, seed `42` (unstratified) |
| **Sample default rate** | `0.94–0.95%` |
| **Post-filter population** | `100,000` rows (no rows excluded by the target definition) |

* **Scope & Sampling Notes:**
  * The population covers U.S. single-family mortgages from Fannie Mae's loan tapes and does not include Canadian portfolios, commercial mortgages, or other loan types. Conclusions are illustrative for a Canadian bank's portfolio, not transferable (`L8` in `FM-CFR-001`).
  * An unstratified random sample (seed `42`) is used; sample target-rate alignment with the population (`0.94–0.95%`) serves as the sample-quality check.

### Unit of analysis — verification status

The pipeline treats each row in `train4.csv` as one loan observation. This assumption has not been verified against the Kaggle data dictionary or the Fannie Mae performance-file glossary. The Fannie Mae source data is a loan-month file, in which each row represents a loan's status in a reporting month. If `train4.csv` preserves that structure, the 100,000-row sample is 100,000 loan-month observations, not 100,000 distinct loans, and the portfolio-size figures in the framework document (the ~$223M portfolio ECL, the "100,000-loan portfolio" language) would need to be recomputed on a loan-level basis. The verification is recorded as Limitation `L11` in the CFR, with the mitigation path described below.

---

## 5. What is the target?

* **Intended Design & Governance Purpose:** Define the target variable, threshold framing, and IFRS 9 staging relationship upfront, and explicitly record any unverified construction logic or threshold mismatches as open governance limitations.
* **Planned / Applied Target Specification:**

| Field | Value |
| :--- | :--- |
| **Target** | `TARGET` (binary) |
| **Threshold / Framing** | Demonstration framing references a `16+` days-past-due threshold (earlier than IFRS 9's `30`-day SICR trigger); `train4.csv` provides `TARGET` and `DPD_INT` |
| **Exclusions** | No rows excluded (`100,000` rows retained) |

* **IFRS 9 Staging & Threshold Alignment Note:**
  * The `16+` day threshold is earlier than IFRS 9's `30`-day rebuttable presumption for Stage 2 (SICR) and typical `90`-day / credit-impaired alignment for Stage 3. Stage 3 in this pipeline is therefore a conservative demonstration proxy (`delinquency_state = default × 5` / `delinquency_state >= 3`), not IFRS 9 Stage 3 (`C6` / `L2` in `FM-CFR-001`).

### Target definition — source and verification status

The pipeline's default target is a binary flag. Its construction from the Fannie Mae source fields is not documented in the pipeline. The Fannie Mae source data uses Current Loan Delinquency Status (`0` = current, `1` = 30–59 days, `2` = 60–89, `3` = 90–119, etc.) and Zero Balance Code (termination reason). The Kaggle derivative `train4.csv` provides `DPD_INT` and `TARGET`, which are derived from these fields by an undocumented construction. The pipeline has not verified whether `TARGET` is (a) a loan-month delinquency flag, (b) a loan-level default flag (did the loan ever reach a credit event?), or (c) a hybrid. This ambiguity is material. If `TARGET` is a loan-month flag, the target rate is a delinquency rate, not a default rate, and the IFRS 9 staging logic (which uses `delinquency_state = default × 5`) is built on a definition that does not match the stated IFRS 9 default convention. If `TARGET` is a loan-level flag, the target rate is a default rate, but the "Current" rows carry `TARGET = 0` even for loans that will default later, which makes the target a current-status flag rather than a default flag. The ambiguity is recorded as Limitation `L12` in the CFR.

---

## 6. What are the constraints?

* **Intended Design & Governance Purpose:** Catalog data, geographic, modeling, verification, and demonstration constraints prior to execution so expectations and governance boundaries remain explicit.
* **Planned / Applied Constraints:**

| Constraint | What It Means |
| :--- | :--- |
| **Public dataset** | Fannie Mae Single-Family Loan Performance; no proprietary bank data |
| **U.S. data only** | No Canadian portfolio validation (`L8`) |
| **No current credit characteristics** | SICR uses a top-decile proxy, not an observed measurement (`L2`) |
| **No direct protected attributes** | Fairness testing is geographic-proxy only (`L7`) |
| **No macro indicators** | Scenario multipliers are analyst-specified, not calibrated to macro variables (`L9`) |
| **Methodology demonstration, not production** | Parameters are specified, not fitted; ECL is illustrative, not a forecast |
| **Target construction not verifiable from the pipeline** | The Kaggle `TARGET` and `DPD_INT` definitions are undocumented in the pipeline; the target's relationship to Fannie Mae's Current Loan Delinquency Status and Zero Balance Code has not been verified. Documented as `L11`/`L12`. |

* **Constraint Governance Note:** These constraints reflect dataset, provenance, and demonstration scope boundaries rather than coding defects; production deployment would require verified data lineage and calibration not present here.

---

## 7. What happens if the model is wrong?

* **Intended Design & Governance Purpose:** Define the financial/analytical consequence of model error, establish automated register guards (`FM-DCR-001`), and specify the compensating mitigants and escalation path.
* **Planned / Applied Guardrail Criteria:**

| Criterion | Threshold | Source |
| :--- | :--- | :--- |
| **AP-02 (LGD band)** | `40%–60%` target; `30%–70%` failure | DCR §3.3 |
| **AP-03 (EAD ratio)** | `60%–75%` target; `50%–85%` failure | DCR §3.3 |
| **AP-04 (ECL ratio of EAD)** | `1%–3%` target; `0.5%–5%` failure | DCR §3.3 |

* **Guardrail Escalation & Traceability:** Success for this demonstration is not "all guards PASS," but that the guards fire as designed—including guards that fire because the data has a known property (`AP-03` fires because the portfolio is young; that is the guard working correctly, not the pipeline failing). When a guard fires outside its target band, the pipeline logs a `WARNING` and records the value in the DCR's results section; when a guard fires outside its failure band, the pipeline logs an `ERROR` and the finding is routed to the Findings Register (`A14`) and the Remediation Plan (`A16`). In this run, `AP-03` fired outside the failure band; the `ERROR` is logged, and Finding `FM-R-08` in the FM Remediation Plan is the trace.

### If the model is wrong — the consequence and the compensation

The model's output is an ECL figure. If the model is wrong, the consequence is a misstated allowance: the bank provisions too little (understating expected losses, overstating capital) or too much (overstating losses, understating distributable earnings). The PDR records three compensating mitigants. First, the model is a demonstration, not a production allowance — the framework document states this, and the approval record (`FM-AR-001`) is a methodology acceptance, not an accounting sign-off. Second, the register guards (`AP-02`, `AP-03`, `AP-04`) fire on parameter values outside their target and failure bands; when a guard fires outside its failure band, the pipeline logs an `ERROR` and the finding is routed to the Findings Register (`A14`) and the Remediation Plan (`A16`). Third, the target and unit-of-analysis ambiguities (`L11`, `L12`) are disclosed in the CFR and the Data Specification; a reader who relies on the ECL figure is told what the figure's target definition does and does not support. The model's wrongness is bounded by these mitigants, not eliminated by them.
