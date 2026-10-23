# Fannie Mae IFRS 9 ECL Demonstration — Risk & Validation Plan

**Document ID:** FM-RVP-001  
**Version / Status:** 1.4 / Frozen at NB03 v1.3 execution (run_id `20261021_121550`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Risk & Validation Plan (RVP)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-DCR-001 · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan) · FM-AR-001 (Approval Record)

---

## At a glance

| Field | Value |
|---|---|
| **Validation type** | **Methodology validation**, not model validation — the pipeline constructs a parameterization; it does not train a model |
| **Verdict location** | The MAP framework document (MAP-STF-FM-001) — **not** in any notebook |
| **Prescribed evaluation** | Register guards AP-02 (LGD), AP-03 (EAD ratio), AP-04 (ECL ratio) |
| **Prescribed reproducibility** | Seed 42; content hash; per-artifact hash in the manifest; audit log per run |
| **Prescribed authority** | Methodology Owner is the only role with sign-off authority |
| **Where results live** | FM-DCR-001 §4 (measured values against criteria) and MAP-STF-FM-001 §7 (framework observations) — **not in this plan** |

> **Document purpose.** This plan defines how the Fannie Mae IFRS 9 ECL Demonstration will be validated: what is in scope, what risks are being tested, what evidence will be produced, who has authority to sign off, and what happens if the validation finds the methodology unfit.
>
> **On the difference from Lending Club.** This is a **methodology validation**, not a model validation. The pipeline does not train a model — it constructs a parameterization (PD, LGD, EAD) and computes ECL. The validation asks: *"Is this parameterization defensible as a methodology demonstration?"* — not *"Is this model fit for deployment?"* The consequence: **there is no verdict in the pipeline.** The verdict lives in the framework document (MAP-STF-FM-001), not in any notebook.

---

## 1. Purpose and scope

This plan defines how the Fannie Mae IFRS 9 ECL Demonstration will be validated: what is in scope, what risks are being tested, what evidence will be produced, who has authority to sign off, and what happens if the validation finds the methodology unfit.

**The critical difference from Lending Club:** this is a **methodology validation**, not a model validation. The pipeline does not train a model — it constructs a parameterization (PD, LGD, EAD) and computes ECL. The validation asks *"Is this parameterization defensible as a methodology demonstration?"* — not *"Is this model fit for deployment?"*

**The consequence:** **there is no verdict in the pipeline.** The verdict lives in the framework document (MAP-STF-FM-001), a separate document authored by the validator. This plan is the contract for that validation — it specifies what the validator will test, what success looks like, and who decides.

**In scope:** parameter construction and freezing, IFRS 9 staging logic, per-stage and probability-weighted ECL, register guard behavior (AP-02/AP-03/AP-04), reproducibility, regulatory mapping.

**Out of scope:** production ECL calculation, capital adequacy, recovery-data LGD calibration, macroeconomic overlay, Canadian portfolio validation, fairness testing (no protected attributes in data).

---

## 2. Validation objectives

The validation has five objectives, in order of priority. **These are validation activities** — what the validator will *do* — not model properties.

| # | Objective | Question it answers | Evidence |
|---|---|---|---|
| 1 | **Parameter defensibility** | Are the PD/LGD/EAD parameter values justified and auditable? | `IFRS9Config`, `notebook_03_parameters.json`, FM-FDL-001 |
| 2 | **Register guard behavior** | Do AP-02/AP-03/AP-04 fire as expected on this portfolio? | NB03 audit log, register guard results |
| 3 | **IFRS 9 staging logic** | Is Stage 1/2/3 classification defensible? | NB03 stage distribution, SICR proxy documentation |
| 4 | **ECL computation** | Are per-stage and probability-weighted ECL correct? | NB03 ECL results, NB04 scenario analysis |
| 5 | **Reproducibility** | Can a reviewer reproduce the results from the artifacts? | Content hashes, manifests, audit logs |

**The ordering is deliberate.** Parameter defensibility is first because the whole methodology rests on the constructed parameters. Register guards are second because they are the primary evidence the pipeline is behaving as designed. Staging logic is third because it is the IFRS 9-specific contribution. ECL computation is fourth because it is arithmetic given the parameters. Reproducibility is fifth because it is the evidence trail.

**Note the absence of a "performance" objective.** Unlike Lending Club, there is no model to evaluate — the parameters are specified, not fitted. The closest equivalent is **parameter defensibility** — are the values justified? The answer is in FM-FDL-001 §5 (constructed parameters) and in FM-DS-001 §8 (limitations L1–L10).

---

## 3. Validation scope

### 3.1 — What is validated

| Dimension | Scope | Notebook |
|---|---|---|
| Data quality | Missingness, sampling reproducibility, content hash | NB01 |
| Parameter construction | `pd_estimate`, `lgd_final`, `ead_final` formulas | NB01, NB03 |
| Parameter freezing | `IFRS9Config` values, `notebook_03_parameters.json` | NB03 |
| Staging logic | Stage 1/2/3 classification, SICR proxy | NB03 |
| ECL computation | Per-stage and probability-weighted ECL | NB03, NB04 |
| Register guards | AP-02 (LGD), AP-03 (EAD), AP-04 (ECL) | NB03 |
| Scenario analysis | Four scenarios, probability weights | NB04 |
| Reproducibility | Content hash, manifest, audit trail | All notebooks |
| Regulatory mapping | OSFI E-23, IFRS 9, Basel III | NB05 |

### 3.2 — What is not validated

| Item | Reason | Documented as |
|---|---|---|
| Production ECL | No deployed service exists | Out of scope |
| Capital adequacy | ECL/CET1 is a scale mismatch | FM-DS-001 §8 L10 (reframed as ECL % of assets) |
| Recovery-data LGD | No recovery data in dataset | FM-DS-001 §8 L3 (constructed LGD) |
| Macroeconomic overlay | No macro factors in dataset | FM-DS-001 §8 L9 (scenario assumptions) |
| Canadian applicability | U.S. data only | FM-DS-001 §8 L8 (limitation) |
| Fairness testing | No protected attributes in dataset | FM-DS-001 §8 L7 (limitation) |
| True SICR | No current credit characteristics | FM-DS-001 §8 L2 (top-decile proxy) |

**The exclusions define the methodology's honest scope.** A validator who reads only §3.1 would think the parameterization is comprehensively validated. The exclusions in §3.2 make the scope defensible.

---

## 4. Risk assessment

### 4.1 — Inherent risks — what the validation will look for

These are the risks the methodology may carry, which the validation is designed to detect. **The validation tests for each; it does not assume the outcome.** The measured findings are in FM-DCR-001 §4 and MAP-STF-FM-001 §7.

| Risk ID | Inherent risk | Why it matters | How the validation detects it |
|---|---|---|---|
| R-01 | **Constructed parameter drift** — `pd_estimate`/`lgd_final` values change | Invalidates ECL | Frozen in `IFRS9Config`; content hash records output |
| R-02 | **PD cap masks tail convexity** — 20% cap understates tail ECL | ECL understated pre-cap | L6 documented; NB04 quantifies pre-cap understatement |
| R-03 | **SICR proxy is by construction** — Stage 2 is top-decile, not true SICR | Stage 2 illustrative | L2 documented; F-07 in findings register |
| R-04 | **Stage 3 collapses to default** — `delinquency_state = default × 5` | Stage 3 not delinquency-driven | C6 documented; F-07 transparency note |
| R-05 | **Young-portfolio EAD** — EAD ratio outside band | AP-03 fires; expected | Documented as expected; not a defect |
| R-06 | **CCF misapplication** — nonzero CCF would be a specification violation | Invalidates EAD | Three assertions (config, computation, return) |
| R-07 | **Reproducibility break** — different run produces different results | Audit failure | Seed 42, content hash, manifest |
| R-08 | **U.S. data as Canadian proxy** — not validated for Canada | Jurisdictional risk | L8 documented; validate on Canadian data |
| R-09 | **No fairness testing** — cannot test for disparate impact | Fairness unverified | L7 documented; acquire HMDA-equivalent data |
| R-10 | **Scenario assumptions uncalibrated** — 45/30/15/10 weights and 1.0–4.5× multipliers are analyst-specified | ECL uncertainty | L9 documented; calibrate against 2008 actuals |

> **On "present" vs "tested."** This register lists the risks the validation will test. It does not state whether each risk materialised — that is FM-DCR-001 §4's job. A validation plan that says "R-02 is present" before the validation has run is a results summary, not a plan. The "Present" language was in the v1.3 register; it has been removed.

### 4.2 — Validation risks — what could go wrong with the validation itself

Separate from the methodology's inherent risks, the validation process itself carries risks.

| Risk ID | Validation risk | Impact on the validation | Mitigation |
|---|---|---|---|
| VR-1 | **Register guard misinterpreted** — a fired guard treated as a defect, not as a signal | Analyst over-corrects a parameter that is inside its rationale | FM-MD-001 §5.3 "detection, not correction" discipline |
| VR-2 | **SICR proxy read as observed SICR** | Stage 2 fraction misinterpreted | FM-MD-001 §5.2 "what a reader must not read" caveat; FM-DS-001 §3 note on `delinquency_state` |
| VR-3 | **PD cap effect understated** | Tail ECL read as precise | NB04 quantifies the pre-cap understatement; L6 discloses |
| VR-4 | **CCF assertion bypassed** | EAD wrong without a guard firing | Three assertions (config, computation, return) |
| VR-5 | **Verdict authored outside the framework document** | Unauditable verdict | Claim-authorization discipline; every status field is an `[ANALYST: ...]` placeholder |

> **Why this register is separate from §4.1.** The methodology's inherent risks are what the validation is *testing*; the validation's own risks are what could make the validation *unreliable*. A single register that mixes them cannot distinguish "the methodology might be uncalibrated" from "the validation might be misreading the calibration."

### 4.3 — Residual risk acceptance conditions

The residual risk profile **after documentation** depends on the framework document's observations and the Methodology Owner's decision. This plan cannot forecast the residual risk state — it can only state the **conditions under which residual risk would be acceptable**.

| Risk ID | Residual risk acceptance condition | Tracking |
|---|---|---|
| R-01 | **Parameters frozen** — `IFRS9Config` unchanged; content hash recorded | FM-DCR-001 §4 |
| R-02 | **PD cap effect disclosed** — pre-cap tail ECL quantified in the framework document | MAP-RP-001 · FM-R-10; MAP-STF-FM-001 §7 |
| R-03 | **SICR proxy disclosed** — Stage 2 presented as "SICR-by-construction," not observed SICR | FM-MD-001 §5.2; MAP-STF-FM-001 §3 |
| R-04 | **Stage 3 limitation disclosed** — C6 limitation stated in every downstream report | FM-DS-001 §3; FM-MD-001 §5.2 |
| R-05 | **AP-03 documented as expected** — EAD ratio flagged and explained (young portfolio) | FM-DCR-001 §4; MAP-STF-FM-001 §7 |
| R-06 | **CCF = 0 enforced** — three assertions pass | NB03 audit log |
| R-07 | **Reproducibility verified** — content hash recorded; manifest complete | FM-DCR-001 §4 |
| R-08 | **U.S. data limitation disclosed** — every framework result labelled illustrative for Canada | MAP-STF-FM-001 §1 |
| R-09 | **Fairness limitation disclosed** — "not tested" in every report | FM-MD-001 §6.1; MAP-STF-FM-001 §7 |
| R-10 | **Scenario assumptions disclosed** — weights and multipliers labelled analyst-specified | FM-MD-001 §4; MAP-STF-FM-001 §5 |

**On the relationship between this plan and the Remediation Plan.** This section states the *conditions* under which residual risk would be acceptable. MAP-RP-001 tracks *whether* the conditions are met — owners, timelines, evidence. A reader who wants to know "is the PD cap disclosed?" reads the framework document; a reader who wants to know "when will the PD cap be recalibrated?" reads MAP-RP-001 · FM-R-10.

---

## 5. Validation methodology

### 5.1 — What the validation will test — methods and criteria

| Activity | Notebook | Evidence artifact | Prescribed acceptance criterion |
|---|---|---|---|
| Parameter construction | NB01 | FM-FDL-001 formulas, `pd_estimate`, `lgd_final`, `ead_final` | All formulas documented in FM-FDL-001 |
| Parameter freezing | NB03 | `IFRS9Config` dataclass, `notebook_03_parameters.json` | All parameters in config; JSON contract written |
| CCF enforcement | NB03 | Three assertions (config, computation, return) | All three pass |
| Register guard behavior | NB03 | Audit log entries for AP-02, AP-03, AP-04 | Guards fire as specified (AP-03 fires; AP-02/AP-04 do not) |
| Staging logic | NB03 | Stage distribution, SICR proxy documentation | Stage 1/2/3 documented; C6 and L2 disclosures present |
| ECL computation | NB03, NB04 | Per-stage ECL, weighted ECL | ECL arithmetic verifiable |
| Scenario analysis | NB04 | Four scenarios with weights and multipliers | Weights sum to 1.0; multipliers documented |
| Reproducibility | All | Content hash, manifest, audit trail | Content hash recorded; manifest complete |
| Regulatory mapping | NB05 | OSFI E-23, IFRS 9, Basel III mapping | All applicable principles mapped |

> **Where the measured results live.** The rightmost column states the *acceptance criterion*. The *measured result* against each criterion — and the pass/fail determination — are recorded in **FM-DCR-001 §4** (measured thresholds) and in **MAP-STF-FM-001 §7** (framework observations). A reader who wants to know "did the methodology meet its criteria?" reads those documents; a reader who wants to know "what were the criteria?" reads this table.

### 5.2 — Success criteria are in FM-DCR-001 §4, not in this plan

The v1.3 plan had a §5.2 "Success criteria" table with a "Current" column (44.28%, 86.95%, 1.09%) and a "Pass/Fail" column. **That table is a results summary, not a pre-registered criterion.** The measured values belong in FM-DCR-001 §4, where they are compared against the criteria. This plan's §5.1 states the criteria (above); FM-DCR-001 §4 states the results.

**A note on the "AP-03 is expected" distinction.** The v1.3 plan said "the methodology passes 7 of 8 criteria. The one failure (AP-03) is expected and documented." That framing is correct, but it belongs in the framework document's observations, not in the validation plan. The plan states the criterion (AP-03's EAD ratio must be in [50%, 85%]); the framework document states the observation (AP-03 fired; the young-portfolio effect explains it).

### 5.3 — The parameter-provenance chain

Unlike Lending Club, the Fannie Mae pipeline does not have a **verdict-ownership chain**. It has a **parameter-provenance chain**. The verdict lives in the framework document, not in a notebook. The pipeline's "contract" is the `notebook_03_parameters.json` artifact — it records what was computed, not whether it is acceptable.

NB01 → constructs parameters (pd_estimate, lgd_final, ead_final)
↓
NB03 → freezes parameters in IFRS9Config, writes notebook_03_parameters.json
↓
NB04 → reads parameters, computes scenarios and fairness
↓
NB05 → reads parameters, produces compliance report
↓
NB06 → reads parameters, exports to Power BI
↓
Framework document (MAP-STF-FM-001) → authors the observations on the methodology
↓
Approval Record (FM-AR-001) → records the Methodology Owner's decision

**The chain is a parameter-provenance chain, not a verdict-provenance chain.** Each notebook asserts its upstream contract (`notebook_03_parameters.json` for NB04–NB06), but the verdict is authored outside the pipeline.

**On the methodological difference between the two projects.** Lending Club validates a **model** — the verdict is in the pipeline (NB03) because the model lives in the pipeline. Fannie Mae validates a **methodology** — the verdict is in the framework document because the methodology lives in the parameter set, and the framework document is where the parameter set is assessed.

**An interviewer who reads both will ask "why is the verdict in a different place?"** The answer is: because the two projects ask different questions. A **model** has to be validated by an independent notebook (NB03) to prevent the developer from approving their own work. A **methodology** can be validated by an independent author (the framework document's validator) because the parameters are specified, not learned — there is no self-approval risk in the same sense, and the framework document's claim-authorization discipline (no notebook writes a compliance verdict) is the structural safeguard.

---

## 6. Validation governance

### 6.1 — Roles and responsibilities

| Role | Name | Responsibility | Authority |
|---|---|---|---|
| IFRS 9 Modeler | `[ANALYST: name]` | NB01–NB03 ownership; parameter construction and freezing | Implements parameters; cannot self-approve methodology |
| Model Risk Validator | `[ANALYST: name]` | Framework document (MAP-STF-FM-001); independent review | Authors the methodology observations |
| Internal Audit | `[ANALYST: name]` | NB04 ownership; governance review | Documents compliance; does not override validator |
| Methodology Owner | `[ANALYST: name]` | Final sign-off on methodology | Can approve, reject, or request changes (with documented rationale) |
| Data Steward | `[ANALYST: name]` | Custodian of Fannie Mae dataset | Controls data access and versioning |

**Separation of duties.**

- The modeler cannot validate their own methodology (NB03 vs framework document separation).
- The validator cannot approve their own observations (framework document vs Methodology Owner separation).
- Internal Audit documents but does not decide.
- The Methodology Owner is the only role with sign-off authority.

### 6.2 — Decision authority matrix

| Decision | Modeler | Validator | Audit | Methodology Owner |
|---|---|---|---|---|
| Change a parameter | Yes (with version bump) | No | Notify | Review |
| Freeze parameters | Yes | No | No | Approve |
| Write the framework observations | No | Yes | No | Review |
| Override the validation observations | No | No | No | Yes (with rationale) |
| Sign off the methodology | No | No | No | Yes |
| Deploy (if applicable) | Implement | No | No | Authorize |

**The matrix mirrors Lending Club's structure but with different roles.** The IFRS 9 Modeler is the developer equivalent; the Model Risk Validator is the validator equivalent; the Methodology Owner is the MRC Chair equivalent. The separation of duties is the same, but the artifact being approved is a **methodology**, not a model.

---

## 7. Validation schedule and milestones

### 7.1 — Phase plan

| Phase | Activity | Pipeline status | Validation status |
|---|---|---|---|
| Phase 1 — Parameter construction | NB01 + NB02 (data, features, parameters) | **Executed** | — |
| Phase 2 — Parameter freezing | NB03 (`IFRS9Config`, parameters JSON) | **Executed** | **Pending framework review** |
| Phase 3 — Scenario analysis | NB04 (stress, fairness) | **Executed** | **Pending framework review** |
| Phase 4 — Compliance | NB05 (regulatory mapping) | **Executed** | **Pending framework review** |
| Phase 5 — Power BI export | NB06 (dashboard artifacts) | **Executed** | **Pending framework review** |
| Phase 6 — Framework observations | Analyst-authored framework document (MAP-STF-FM-001) | **Drafted** | **Pending signature** |
| Phase 7 — Methodology Owner sign-off | Methodology Owner decision (FM-AR-001) | — | **Pending** |
| Phase 8 — Production handoff | If applicable, hand off to production team | — | Out of scope for this demonstration |

> **On the two status columns.** "Pipeline status" answers "did the notebook run?" "Validation status" answers "has the validation activity been signed off?" The two are different: a notebook can have executed (pipeline status: Executed) without the validation being complete (validation status: Pending framework review). Conflating them — as the v1.3 document did — makes the plan look as if the validation is complete when only the pipeline has run.

### 7.2 — Key milestones

| Milestone | Target date | Owner | Status |
|---|---|---|---|
| Framework observations drafted (MAP-STF-FM-001) | `[ANALYST: date]` | Validator | Pending |
| Methodology Owner review | `[ANALYST: date]` | Methodology Owner | Pending |
| Methodology Owner sign-off (FM-AR-001) | `[ANALYST: date]` | Methodology Owner | Pending |
| Production handoff assessment | `[ANALYST: date]` | Production team | Out of scope for this demonstration |

### 7.3 — Escalation path

| Trigger | Escalation |
|---|---|
| CRITICAL parameter finding | Immediate escalation to Methodology Owner; parameters frozen pending review |
| Register guard fires (AP-02, AP-03, AP-04) | Documented; analyst investigates; not necessarily an escalation |
| Reproducibility break (content hash differs) | Escalate to Methodology Owner; re-run required |
| New limitation discovered | Document in L1–L10 register (FM-DS-001 §8); re-validate if material |
| Methodology Owner overrides validator | Documented rationale required; re-validation on the affected dimension |

**The escalation path is different from Lending Club.** In Lending Club, a CRITICAL finding triggers immediate MRC escalation because deployment is blocked. In Fannie Mae, a CRITICAL parameter finding triggers escalation to the Methodology Owner because the parameters are frozen — changing them is a MAJOR version bump and a full pipeline re-run. **The escalation is about change control, not deployment blocking.**

---

## 8. Validation-process limitations

### 8.1 — On the relationship to other limitations

The methodology's limitations (L1–L10) are in FM-DS-001 §8. The method's open items are in FM-MD-001 §8.2. The framework's observations are in MAP-STF-FM-001 §7. **This section records only the limitations of the validation *process* — things a reader of the framework document should know about the validation itself, not about the methodology or the data.**

### 8.2 — Validation-process limitations

| ID | Limitation | Impact on the validation | Mitigation |
|---|---|---|---|
| VL-1 | **Fairness is not tested** | Cannot verify disparate impact | Documented in FM-MD-001 §6.1 and MAP-STF-FM-001 §7; longer-horizon data acquisition |
| VL-2 | **No time-based stability** | PSI not assessable | Documented in FM-DS-001 §6.2; walk-forward requires time-series data |
| VL-3 | **No recovery-data LGD** | LGD is constructed, not calibrated | Documented in FM-FDL-001 §5.3; longer-horizon data acquisition |
| VL-4 | **No macro overlay** | Economic cycle not captured | Documented in FM-MD-001 §4; scenario multipliers are analyst-specified |
| VL-5 | **SICR proxy is by construction** | Stage 2 illustrative, not observed | Documented in FM-MD-001 §5.2 and MAP-STF-FM-001 §3 |
| VL-6 | **Stage 3 collapses to default** | Not true delinquency staging | C6 limitation documented in FM-DS-001 §3 |
| VL-7 | **PD cap masks tail convexity** | Tail ECL understated pre-cap | NB04 quantifies; MAP-STF-FM-001 §7 discloses |
| VL-8 | **U.S. data only** | Not validated for Canada | Documented in FM-DS-001 §8 L8 |
| VL-9 | **The validation is performed by the same analyst who built the pipeline** | The independence claim rests on the claim-authorization discipline (no notebook writes a verdict) and the decision authority matrix (§6.2), not on organizational separation | The framework document's `[ANALYST: ...]` placeholders and the Methodology Owner's sign-off are the structural safeguards |

> **On VL-9.** The Fannie Mae project is a portfolio artifact, and the same analyst built the pipeline and will write the framework document. The independence claim rests on the **technical** separation (the pipeline writes no verdict; every status field is a placeholder) and the **governance** separation (§6.2's decision authority matrix), not on organizational independence. In a production deployment, the validator would be a separate person in a separate function. The plan states this clearly — a reader who assumes organizational independence has misunderstood the artifact.

### 8.3 — Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **Framework observations** | Pending. The validator must author the observations on whether the methodology is defensible as a demonstration. | Phase 6 in §7.1 |
| 2 | **Methodology Owner sign-off** | Pending. The Methodology Owner (the MRC Chair equivalent) must record the decision and rationale. | Phase 7 in §7.1; FM-AR-001 |
| 3 | **Fairness gap** | L7 documents that fairness is untested. A production implementation would need to address this, but the demonstration scope is parameterization and staging. | FM-R-09 in MAP-RP-001 (longer-horizon) |
| 4 | **PD cap** | L6 documents that the 20% cap masks tail convexity. NB04 quantifies the pre-cap understatement. A production implementation should either remove the cap (with a separate tail-risk analysis) or justify 20% explicitly. | FM-R-10 in MAP-RP-001 |
| 5 | **SICR proxy** | L2 and F-07 document that Stage 2 is top-decile construction, not true SICR. A production implementation would need current credit characteristics. | FM-R-07 in MAP-RP-001 |

**On the routing.** Each open item is routed to a Phase in §7.1 (for items that are part of the validation lifecycle) or to a MAP-RP-001 finding (for items that are remediation work). The routing makes it clear which items are pending the validator's work and which are pending the analyst's work.

---

*End of document.*
