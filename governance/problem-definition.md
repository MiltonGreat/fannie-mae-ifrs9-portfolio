# Fannie Mae IFRS 9 ECL Demonstration — Problem Definition Record

**Document ID:** FM-PDR-001  
**Version / Status:** 0.2 / Draft — reconstructed from implementation  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Problem Definition Record (PDR)  
**Companion documents:** FM-DS-001 (Data Specification) · FM-FDL-001 (Feature Decision Log) · FM-MD-001 (Methodology Document) · FM-RVP-001 (Risk & Validation Plan) · FM-DCR-001 (Decision Criteria Register) · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Business problem** | Build a documented, auditable IFRS 9 ECL methodology demonstration on real mortgage data |
| **Intended use** | Methodology demonstration and training material — **not** a production ECL engine |
| **Explicitly not for** | Capital calculation, regulatory reporting, or any balance-sheet figure |
| **Unit of analysis** | One loan, observed once at the snapshot date |
| **Target** | Binary default — 1 if 90+ days past due at the snapshot, 0 otherwise |
| **Constructed parameters** | `pd_estimate`, `lgd_final`, `ead_final` — constructed, not fitted (see §4.2) |
| **Success criteria** | 3 groups: pipeline (2), documentation (2), methodology (3) — see §5 |
| **Documented limitation** | Constructed parameters are illustrative, not calibrated to recovery data; AP-03 (EAD ratio) fires on a young portfolio |

> **Provenance note.** This PDR was reconstructed after the pipeline was written, from the implementation and the notebook outputs. It is a **frozen historical record of what the project intended**, not evidence of what it achieved. Where the PDR states a criterion, the criterion is what the project set out to meet; whether the methodology met it is recorded in FM-DCR-001 (measured thresholds) and MAP-STF-FM-001 (framework observations). A reconstructed PDR is a legitimate artifact provided it is honest about its provenance; this one is.

---

## 1. Business problem

The IFRS 9 modelling team needs a methodology demonstration: a working, documented end-to-end pipeline that takes historical mortgage loan data, calibrates PD/LGD/EAD, stages loans under IFRS 9 (Stage 1/2/3), computes a probability-weighted Expected Credit Loss, and produces the evidence a regulator or internal audit would expect. **The problem is not "build the best mortgage default model" — it is "show, with real data and auditable artefacts, how an IFRS 9 ECL calculation is constructed and defended."**

**What goes wrong without it:** IFRS 9 implementations get built ad hoc, with undocumented parameter choices (LGD floor, SICR thresholds, scenario weights) that fail audit. This project exists to make those choices explicit, traceable, and challengeable.

---

## 2. Intended use and users

| Role | Description |
|---|---|
| **Primary user** | IFRS 9 modelling team — a reference implementation for methodology development |
| **Secondary user** | Internal audit / model risk — reviewing whether the methodology is defensible |
| **Tertiary user** | Regulators (OSFI E-23 audience) — as a demonstration of documentation and governance discipline |

**Intended use:** Methodology demonstration and training material. **Not** a production ECL engine.

**Explicitly not intended for:** capital calculation, regulatory reporting, or any balance-sheet figure.

---

## 3. Population and unit of analysis

**Population:** U.S. single-family mortgages in the Fannie Mae Single-Family Loan Performance dataset (2021–2023 vintages in this sample).

**Unit of analysis:** **One loan, observed once at the snapshot date.** The dataset is a cross-section, not a panel — each loan appears once, with `AGE` recording months since origination at the snapshot date. A reader who assumes the data is loan-month panel data will misread the per-loan aggregates (e.g., the default rate is a snapshot rate, not a cumulative-window default rate).

**Sample:** 100,000 rows randomly drawn from 1,672,822 source rows (seed 42). The sample is a cross-section of the source dataset; it is not the full dataset, and the full dataset is used in production (the AS records the sampling method).

---

## 4. Target and constructed parameters

### 4.1 — Target (observed outcome)

**Target variable:** `default` (binary).

| Status | Class |
|---|---|
| 90+ days past due at the snapshot date, or in default per Fannie Mae's status field | **1 — default** |
| All performing statuses at the snapshot date | **0 — performing** |
| Unknown status | Excluded from the ECL calculation |

**Threshold:** 90+ days, consistent with IFRS 9 Stage 3 and the FFIEC/OCC standard for mortgage default.

**Observation window:** **A snapshot classification at the observation date.** A loan is classified as default if it is 90+ days past due *at the snapshot date*, not if it was ever 90+ days past due at any point in its history. The target is a point-in-time default indicator, not a cumulative-window default indicator. This is a deliberate scope choice: the Fannie Mae dataset provides a snapshot, and the pipeline does not have access to a full payment history per loan that would support a cumulative-window classification. The distinction matters for interpreting the default rate — a snapshot default rate is lower than a cumulative-window default rate on the same portfolio, because loans that were delinquent earlier but cured before the snapshot are classified as performing.

### 4.2 — Constructed parameters (inputs to the ECL calculation)

**These are not targets.** They are constructed inputs that the ECL calculation uses. The distinction matters: a reader who looks for the target should find `default` (§4.1), not `pd_estimate`.

| Parameter | Construction | Range | Built in |
|---|---|---|---|
| `pd_estimate` | Weighted combination of FICO, LTV, DTI, AGE; normalized and scaled | [0.01, 0.20] (clipped) | NB01 |
| `lgd_final` | Base 45% + age / LTV / property adjustments | [0.10, 0.90] (clipped) | NB01 (initial), NB03 (final) |
| `ead_final` | `ORG_UPB` × prepayment factor × curtailment factor, with CCF = 0 | Implicit | NB03 |

**The documented limitation:** `pd_estimate` and `lgd_final` are **constructed, not fitted**. The Feature Decision Log (FM-FDL-001) records the formula and every coefficient. A production model would fit these via logistic regression (for PD) and recovery data (for LGD). Here they are specified because the goal is methodology demonstration, not calibration quality.

**A note on the parameters' role in the verdict.** Because the parameters are constructed, the framework's results (ECL, coverage) are **illustrative of the methodology**, not indicative of any portfolio's expected loss. This is the framework's most important scope statement (see MAP-STF-FM-001 §1).

---

## 5. Success criteria

The success criteria are grouped by what "success" means for which aspect of the project. **The measured results against each criterion are in FM-DCR-001 (measured thresholds) and MAP-STF-FM-001 (framework observations). They are not in this PDR** — a PDR that contains its own results cannot demonstrate that the criteria were pre-registered.

### 5.1 — Pipeline success (did the pipeline behave as specified?)

| Criterion | Target | Source | Where the result is recorded |
|---|---|---|---|
| Pipeline reproducibility | Two runs with seed 42 produce identical content hashes | OSFI B-13 §3.2 | FM-DCR-001 §4 (content hash comparison) |
| Content hash | `final_clean_content_hash` recorded in the manifest | OSFI B-13 §3.2 | FM-DCR-001 §4 (manifest entry) |

### 5.2 — Documentation success (is the methodology documented and auditable?)

| Criterion | Target | Source | Where the result is recorded |
|---|---|---|---|
| Parameter documentation | Every parameter in `IFRS9Config` has a documented rationale | OSFI E-23 §4.2 | FM-MD-001 §4 and FM-DCR-001 §4 |
| Claim authorization | No notebook writes a compliance verdict; the analyst does | OSFI E-23 §4.2 | FM-DCR-001 §4 and MAP-STF-FM-001 §7 |

### 5.3 — Methodology success (does the methodology demonstrate what it claims?)

| Criterion | Target | Source | Where the result is recorded |
|---|---|---|---|
| ECL calculation transparency | Per-stage and probability-weighted ECL both computed and reconcilable | IFRS 9 §5.5.17–18 | FM-DCR-001 §4 and MAP-STF-FM-001 §5 |
| CCF discipline | CCF = 0 asserted at every boundary (config, computation, return) | Basel III | FM-DCR-001 §4 |
| Register guards | AP-02 (LGD band), AP-03 (EAD ratio), AP-04 (ECL ratio) fire on drift | FM-DCR-001 | FM-DCR-001 §4 and MAP-STF-FM-001 §7 |

> **On the "measured on current run" paragraph.** The v0.1 PDR reported "all criteria met except AP-03" in the body of the success criteria section. That paragraph is removed. The measured results are in FM-DCR-001 §4 (for each criterion) and in MAP-STF-FM-001 §7 (for the framework observations, including AP-03). A reader who wants to know "did the methodology meet its criteria?" reads those documents; a reader who wants to know "what were the criteria?" reads this PDR.

---

## 6. Scope and exclusions

### In scope

- PD / LGD / EAD calibration from historical data (constructed, not fitted)
- IFRS 9 Stage 1/2/3 classification (with documented SICR proxy)
- Probability-weighted ECL across four scenarios
- Limitations register (L1–L10)
- Regulatory mapping against OSFI E-23, IFRS 9, Basel III
- Reproducibility manifest

### Excluded

| Exclusion | Reason |
|---|---|
| Production scoring | No deployed scoring service exists |
| Capital adequacy calculation | ECL ratio of EAD is a coverage diagnostic, not a capital measure |
| Canadian portfolio validation | U.S. data only |
| Direct recovery-data LGD calibration | LGD is constructed from assumptions, not observed recoveries |
| Full macro overlay | No macro factors in dataset; scenario multipliers are analyst-specified |
| Validation Opinion | The Fannie Mae notebook produces evidence; the verdict is authored separately (in the MAP) |

---

## 7. Key assumptions and dependencies

### 7.1 — Assumptions

- **The 100,000-row sample is representative** of the full Fannie Mae dataset for methodology demonstration purposes.
- **90+ days past due is the correct default threshold** for IFRS 9 Stage 3.
- **Term mortgages have CCF = 0** (no undrawn commitment — Basel III scope).
- **The SICR proxy** (highest-risk decile of non-Stage-3 loans) is a reasonable stand-in for true SICR in the absence of current credit characteristics. **A reader must not read "Stage 2 = 9.9%" as an observed SICR rate** — see MAP-STF-FM-001 §3 for the mechanism.
- **Scenario weights (45/30/15/10) and multipliers (1.0/1.8/3.0/4.5× PD)** are illustrative, not calibrated to history.
- **`pd_estimate` and `lgd_final` are illustrative constructions**, not calibrated parameters.

### 7.2 — Dependencies

| Category | Dependency |
|---|---|
| **Data** | Fannie Mae ZIP → `train4.csv` (reproducible sampling at seed 42) |
| **Upstream documents** | FM-FDL-001 (feature decisions), FM-MD-001 (methodology), FM-AS-001 (analytical spec), FM-DCR-001 (decision criteria) |
| **Notebooks** | NB01 → NB02 → NB03 → NB04 → NB05 → NB06 (in order; each asserts its upstream contracts) |
| **External tools** | `pandas`, `numpy`, `scipy`, `statsmodels` |
| **Seed** | **One project-wide seed (`RANDOM_STATE = 42`)**, used for both the sample draw (NB01) and the cross-validation (NB02+). The same value in both places is a project convention (carried from the Lending Club pipeline), not two independent seeds. |
| **Parameters contract** | `notebook_03_parameters.json` is the frozen contract for NB04 and NB05 |

**On the seed convention.** The v0.1 PDR said "two seeds: one for the sample draw and one for cross-validation, both at 42." That phrasing implies two distinct seeds that happen to have the same value. The v0.2 PDR clarifies: **one project-wide seed (42), used in two places.** Changing it invalidates both the sample and the cross-validation, which is the same failure mode in both places. If a future version wants two independent seeds, that is a methodology change, not a documentation clarification — it belongs in FM-MD-001 §4 and a corresponding FM-PDR-001 revision.

---

## 8. Approval and ownership

| Role | Name | Responsibility |
|---|---|---|
| IFRS 9 Modeler | `[ANALYST: name]` | NB01–NB03 ownership; methodology design |
| Model Risk Validator | `[ANALYST: name]` | Independent review (MAP framework document, MAP-STF-FM-001) |
| Internal Audit | `[ANALYST: name]` | Governance review (NB04 compliance report) |
| Methodology Owner | `[ANALYST: name]` | Final sign-off on methodology |
| Data Steward | `[ANALYST: name]` | Custodian of the Fannie Mae dataset |

**Approval workflow:** Modeler → Validator (independent review) → Internal Audit (compliance review) → Methodology Owner (sign-off).

> **On the approval's meaning.** The approval is of the **methodology**, not of a deployable model. The artefact is a demonstration; the Methodology Owner's sign-off accepts the methodology as a demonstration, subject to the documented limitations. This is a narrower acceptance than a model deployment approval — a reader who conflates the two will overstate what the approval means. The MAP's reading guide (when it is drafted) is where the Lending Club approval (model deployment) and the Fannie Mae approval (methodology acceptance) are distinguished. This PDR does not make the cross-project comparison; it states the Fannie Mae approval's meaning.

> **Note on the placeholders.** The `[ANALYST: name]` fields are the analyst's contribution, not omissions. The PDR produces the structure and the roles; the analyst (or the project owner) assigns the names and records the approval.

---

## 9. What this PDR does not decide

This PDR fixes the problem, the intended use, the population, the target, the constructed parameters, the pre-registered success criteria, and the scope. It does **not** decide:

| Question | Where the answer lives |
|---|---|
| **Did the pipeline meet its success criteria?** | FM-DCR-001 §4 (measured thresholds) and MAP-STF-FM-001 §7 (framework observations) |
| **Is the methodology accepted?** | FM-AR-001 (Approval Record) — the Methodology Owner's decision |
| **Is the ECL calculation defensible?** | MAP-STF-FM-001 (framework document) — the observations for the reader |
| **What are the limitations?** | FM-MD-001 §8 (methodology limitations), FM-DS-001 §8 (data limitations), and MAP-STF-FM-001 §7 (framework observations) |
| **What remediation is required?** | MAP-RP-001 (Remediation Plan) — the Fannie Mae findings FM-R-06 through FM-R-10 are tracked there |

**The separation is deliberate.** A PDR that also contains the verdict is not a PDR; it is a PDR with a framework document stapled to it. This PDR ends at the pre-registration. Everything downstream is in the documents listed above.

---

*End of document.*
