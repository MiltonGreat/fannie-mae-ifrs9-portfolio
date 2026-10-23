# Fannie Mae IFRS 9 ECL Demonstration — Decision Criteria Register

**Document ID:** FM-DCR-001  
**Version / Status:** 1.4 / Frozen at NB03 v1.3 execution (run_id `20261021_121550`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Decision Criteria Register (DCR)  
**Frozen at:** NB03 v1.3 execution (run_id `20261021_121550`)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan) · FM-AR-001 (Approval Record)

---

## At a glance

| Field | Value |
|---|---|
| **Purpose** | Define every decision criterion in the Fannie Mae pipeline: parameter bounds, staging thresholds, register guard bands, scenario weights, reproducibility requirements |
| **Criterion families** | §3.1 Parameter bounds · §3.2 Staging thresholds · §3.3 Register guards · §3.4 Scenario criteria · §3.5 Reproducibility |
| **This run's guard results** | AP-02 PASS · **AP-03 OUTSIDE FAILURE BAND (expected)** · AP-04 PASS |
| **Where results live** | This document (§4 results against criteria) — **the only Fannie Mae governance document where measured values belong** |
| **Authority for threshold changes** | Methodology Owner (sole approver) |
| **Version chain** | v1.3 → v1.4 (this revision — structural; no threshold changes) → v2.0 (on any MAJOR threshold change) |

> **Document purpose.** This register records every decision criterion used in the Fannie Mae pipeline: the register guard thresholds (AP-02, AP-03, AP-04), the parameter bounds (PD cap, LGD floor/cap, CCF), the staging thresholds (Stage 1/2/3 boundaries), the scenario weights and multipliers, and the reproducibility requirements.
>
> **Structure.** The document has four parts: (1) the **criterion catalogue** (§3) — thresholds and bands, no results; (2) the **results against criteria** (§4) — measured values, pass/fail, and status per guard; (3) the **verdict record** (§5) — the methodology's status statement; (4) the **governance** (§6–§9) — change control, authority, open items. A reader who wants the threshold reads §3; a reader who wants the result reads §4; a reader who wants the status reads §5.

> **The distinction from Lending Club's DCR.** Lending Club's DCR is organized around **model performance criteria** (AUC, KS, DI, HL). Fannie Mae's DCR is organized around **parameter defensibility** (are the values justified?) and **register guard behavior** (do the automated checks fire as expected?). The two registers have different criterion families because the two projects ask different questions — one is a model validation, the other is a methodology validation.

---

## 1. Purpose and scope of the register

This register records every decision criterion used in the Fannie Mae pipeline: the register guard thresholds (AP-02, AP-03, AP-04), the parameter bounds (PD cap, LGD floor/cap, CCF), the staging thresholds (Stage 1/2/3 boundaries), the scenario weights and multipliers, and the reproducibility requirements.

**The distinction from Lending Club.** Lending Club's DCR is organized around model performance criteria (AUC, KS, DI, HL). Fannie Mae's DCR is organized around **parameter defensibility** (are the values justified?) and **register guard behavior** (do the automated checks fire as expected?). The two registers have different criterion families because the two projects ask different questions.

**The distinction from the framework document (MAP-STF-FM-001).** This DCR defines the criteria and reports the measured results against them. The framework document (MAP-STF-FM-001) interprets the results and states the observations. The DCR is the *standard*; the framework document is the *assessment*. A reader who wants to know "what was the criterion?" reads this DCR; a reader who wants to know "what does the result mean?" reads the framework document.

**The distinction from the RVP (FM-RVP-001).** The RVP plans the validation — what will be tested and who decides. This DCR is the criterion catalogue plus the results. The RVP cites the DCR for the criteria; the DCR reports the results against them.

---

## 2. Criterion catalogue overview

The Fannie Mae DCR contains five criterion families:

| Family | Criteria | Source |
|---|---|---|
| §3.1 — Parameter bounds | PD floor/cap, LGD floor/cap, CCF, EIR, prepayment, curtailment | DCR §1 |
| §3.2 — Staging thresholds | Stage 1/2 boundaries, SICR triggers | DCR §2 |
| §3.3 — Register guards | AP-02 (LGD), AP-03 (EAD), AP-04 (ECL) | DCR §3 |
| §3.4 — Scenario criteria | Scenario weights, PD/LGD multipliers | DCR §4 |
| §3.5 — Reproducibility | Content hash, seed, manifest | DCR §5 |

**The families are parameter-focused, not performance-focused.** This reflects the methodological difference: Fannie Mae is not evaluating a model — it is validating a **construction**. The parameters are analyst-specified, so the criteria are about whether the construction is defensible, not whether a fitted model performs.

---

## 3. Criterion catalogue — thresholds and bands

**This section defines the criteria and their bands. It does not report this run's results** — the results are in §4. A reader who wants to know "what is the PD cap?" reads §3.1; a reader who wants to know "did the lifetime PD hit the cap?" reads §4.

### 3.1 — Parameter bounds

| Parameter | Value | Rationale | Source |
|---|---|---|---|
| PD floor | 0.0001 (0.01%) | Prevent zero PD | DCR §1 |
| PD cap | 0.20 (20%) | Masks tail convexity | DCR §1 (L6) |
| LGD floor | 0.10 (10%) | Prevent zero LGD | DCR §1 |
| LGD cap | 0.90 (90%) | Prevent LGD > 100% | DCR §1 |
| CCF (term mortgage) | 0.0 | No undrawn commitment (Basel III) | DCR §1 |
| EIR | 0.035 (3.5%) | Industry average | DCR §1 (L5) |
| Prepayment rate | 0.08 (8%) | Historical average | DCR §1 (L4) |
| Curtailment rate | 0.02 (2%) | Historical average | DCR §1 (L4) |

**The two most consequential bounds:**

**`pd_cap = 0.20` — the PD cap fires on a lifetime PD above 20%.** This is the mechanism behind **L6**: the PD cap masks tail convexity. The DCR documents the cap as a deliberate stability measure, not an accident. The cap's rationale and its per-run effect are in §4.1 and in MAP-STF-FM-001 §7.

**`ccf = 0.0` — asserted at three boundaries.** The DCR says CCF is 0 for term mortgages — **any nonzero CCF is a specification violation**. This is the single most important DCR rule in the Fannie Mae pipeline. The three assertions (config, computation, return) are documented in FM-MD-001 §4.2.

**The DCR is where these values live.** Changing any of them requires a **MAJOR** version bump and a full pipeline re-run (see §6).

### 3.2 — Staging thresholds

| Criterion | Value | Source |
|---|---|---|
| Stage 1 threshold | 0.005 (0.5%) | DCR §2 |
| Stage 2 threshold | 0.05 (5%) | DCR §2 |
| SICR PD increase | 2.0× | DCR §2 |
| SICR FICO drop | 50 points | DCR §2 |
| SICR delinquency backstop | 30 DPD | DCR §2 |

**The SICR thresholds are the DCR's most interpretive content.** The 2× PD increase, 50-point FICO drop, and 30 DPD backstop are all analyst-specified — they are "reasonable and supportable" under IFRS 9, not empirically calibrated. The DCR records them as such. **A reader must not read them as calibrated thresholds** — they are analyst judgments, documented and challengeable.

**The staging mechanism is documented in FM-MD-001 §5.2.** In summary: Stage 3 is driven by the target (`delinquency_state = default × 5`, the C6 limitation); Stage 2 is top-decile proxy SICR (L2). The DCR records the *thresholds*; the MD records the *mechanism*. A reader who wants the mechanism reads the MD.

### 3.3 — Register guard bands

**AP-02 — LGD band:**

| Criterion | Value | Source |
|---|---|---|
| Target band | 40%–60% | DCR §3 AP-02 |
| Failure band | 30%–70% | DCR §3 AP-02 |

**AP-03 — EAD ratio band:**

| Criterion | Value | Source |
|---|---|---|
| Target band | 60%–75% | DCR §3 AP-03 |
| Failure band | 50%–85% | DCR §3 AP-03 |

**AP-04 — ECL ratio of EAD:**

| Criterion | Value | Source |
|---|---|---|
| Target band | 1%–3% | DCR §3 AP-04 |
| Failure band | 0.5%–5% | DCR §3 AP-04 |

**The AP-03 rule is the DCR's most important contribution.** It says: **if the EAD ratio is outside the band, check CCF first, then investigate the portfolio.** On a young portfolio, the ratio is high because prepayment and curtailment have not had time to reduce EAD. The DCR makes this a designed outcome, not a silent failure — the guard fires, the analyst investigates, and the finding is documented.

### 3.4 — Scenario criteria

| Scenario | Weight | PD multiplier | LGD multiplier | Source |
|---|---|---|---|---|
| Baseline | 0.45 | 1.0× | 1.0× | DCR §4 |
| Adverse | 0.30 | 1.8× | 1.2× | DCR §4 |
| Severely Adverse | 0.15 | 3.0× | 1.5× | DCR §4 |
| Tail Risk | 0.10 | 4.5× | 2.0× | DCR §4 |

**The weights sum to 1.0** — this is a hard requirement, checked at runtime by the pipeline's scenario aggregation. **The multipliers are analyst-specified** based on 2008 historical analysis (L9). The DCR documents them as assumptions, not calibrated parameters.

### 3.5 — Reproducibility criteria

| Criterion | Value | Source |
|---|---|---|
| Random seed | 42 | DCR §5 (AS-001 §8) |
| Content hash | SHA-256-16 | DCR §5 |
| Manifest | Per-notebook, per-file | DCR §5 |
| Audit log | Per-run, per-notebook | DCR §5 |

**The reproducibility criteria are the DCR's evidence trail.** Every artifact carries a `run_id` and `notebook_version`; every run produces an audit log; the content hash records the cleaned dataframe. A reviewer can reproduce the results from the artifacts alone.

---

## 4. Results against criteria — this run

This section reports the measured values against the criteria in §3, the pass/fail determination per guard, and the resulting status. **The measured values are here, not in §3** — a reader who wants to know "what is the EAD ratio band?" reads §3.3; a reader who wants to know "did the EAD ratio fire?" reads this section.

### 4.1 — Register guard results

| Guard | Measured | Band status | Status |
|---|---|---|---|
| AP-02 (LGD) | 44.28% | Inside target band (40–60%) | **PASS** |
| AP-03 (EAD) | 86.95% | **Outside failure band (50–85%)** | **OUTSIDE FAILURE — expected** |
| AP-04 (ECL) | 1.09% | Inside target band (1–3%) | **PASS** |

**On AP-03's "expected" status.** The 86.95% EAD ratio is outside the 50–85% failure band. **The DCR says this is expected on a young portfolio** — the average loan age is 16 months, mostly 2021–2023 vintages, so prepayment and curtailment have not had time to reduce EAD. **The guard fires as designed**; the analyst documents the cause; it is not a methodology defect, it is a feature of the data. See FM-DS-001 §6.2 for the "young portfolio" mechanism and MAP-STF-FM-001 §7 for the framework observation.

**On the two-tier structure.** A value outside the target band but inside the failure band triggers investigation (WARNING in the audit log). A value outside the failure band triggers escalation (ERROR in the audit log). AP-03 is outside the failure band, so it logged an ERROR. The analyst must document it — which the DCR does, in this section.

### 4.2 — Parameter results

| Parameter | Measured | Note |
|---|---|---|
| Unconditional PD | 0.95% | Sample default rate; clipped to [0.01%, 20%] (no-op on this run) |
| Lifetime PD | 20.00% (capped) | Pre-cap: ~26.1%; the cap fired (L6) |
| Average LGD | 44.28% | Inside 40–60% target band (AP-02 PASS) |
| Average EAD | $204,745 | EAD ratio 86.95% (AP-03 fired) |
| EAD ratio | 86.95% | Outside 50–85% failure band; expected for young portfolio |
| ECL ratio of EAD | 1.09% | Inside 1–3% target band (AP-04 PASS) |
| CCF | 0.0 | Three assertions passed |

**On the lifetime PD cap.** The pre-cap lifetime PD is `1 - (1 - 0.0095)^30 ≈ 26.1%`. The cap at 20% clips this to 20%. **The cap's effect on Tail Risk ECL** is quantified in MAP-STF-FM-001 §7 — the pre-cap understatement is disclosed as **L6**. A reader who sees "Lifetime PD: 20.00%" should understand that this is the *capped* value, not the model's raw output.

### 4.3 — Staging results

| Stage | Fraction | Mechanism |
|---|---|---|
| Stage 1 | 89.1% | Remaining non-Stage-3 non-top-decile loans |
| Stage 2 | 9.9% | **Top-decile proxy SICR** (L2, F-07) |
| Stage 3 | 1.0% | `delinquency_state >= 3` (i.e., `default × 5`, C6) |

**On the staging fractions.** Stage 2's 9.9% is **SICR-by-construction**, not observed credit deterioration. Stage 3's 1.0% is **target-driven**, not observed delinquency. **A reader must not read either fraction as an empirical measurement** — they are constructions, documented as L2 and C6. See FM-MD-001 §5.2 for the mechanism.

### 4.4 — Scenario results

| Scenario | PD | LGD | ECL (per loan) | Weight | Weighted ECL |
|---|---|---|---|---|---|
| Baseline | 0.95% | 44.28% | $834.70 | 45% | $375.62 |
| Adverse | 1.72% | 53.13% | $1,802.96 | 30% | $540.89 |
| Severely Adverse | 2.86% | 66.41% | $3,756.16 | 15% | $563.42 |
| Tail Risk | 4.29% | 88.55% | $7,512.33 | 10% | $751.23 |
| **Total** | — | — | — | **100%** | **$2,231.16** |

**The weighted total is $2,231.16 per loan.** On a 100,000-loan portfolio, the portfolio ECL is ~$223M. **The portfolio ECL is a demonstration of the arithmetic, not a forecast** — the parameters are constructed, not calibrated, and the results are illustrative (see FM-PDR-001 §4.2).

**On the severity check.** The severely adverse ECL rate versus the 2008–2010 historical average produces a ratio of ~1.27×, **below the DCR's 1.5× target**. This is recorded in the framework document's observations (MAP-STF-FM-001 §7) as an investigation trigger, not as a failure — the comparison is directional, not empirical.

---

## 5. Verdict record — the methodology's status

**The Fannie Mae DCR does not have a single risk score in the same sense as Lending Club.** The methodology's status is the set of register guard results and the parameter documentation completeness. **There is no 0–10 scale because there is no model to score.**

### 5.1 — Status bands per guard

| Guard status | Meaning | Action |
|---|---|---|
| **PASS** | Within target band | No action |
| **OUTSIDE TARGET** | Within failure band but outside target | Document; investigate |
| **OUTSIDE FAILURE** | Outside failure band | Document; escalate if unexpected |

**On this run:** AP-02 PASS, **AP-03 OUTSIDE FAILURE (expected)**, AP-04 PASS.

### 5.2 — The methodology's status statement

**This run's status:** The methodology's parameterization is **consistent with the register guard design**, with one **documented exception** (AP-03, the young-portfolio EAD ratio). The exception is expected, documented in this DCR (§4.1), and explained in the framework document (MAP-STF-FM-001 §7). The methodology's limitations (L1–L10) are documented in FM-DS-001 §8. **The methodology is defensible as a demonstration**, subject to the limitations documented in the framework document.

**What "defensible as a demonstration" means:** the methodology's parameters are documented and auditable, the register guards fire as designed, and the limitations are disclosed. **It does not mean** the methodology is suitable for production — the constructed parameters, the SICR proxy, and the young-portfolio effect are all documented limitations, and a production implementation would need to address them.

**Who decides whether the methodology is accepted:** the Methodology Owner (FM-AR-001). This DCR's §5.2 is the *input* to that decision, not the decision itself. A reader who sees "defensible as a demonstration" here should read it as a summary of the evidence, not as the Methodology Owner's approval.

### 5.3 — The three rating systems

The Fannie Mae DCR uses three distinct rating systems, each with its own meaning:

| System | Scale | Bands | Purpose |
|---|---|---|---|
| Guard status | categorical | PASS / OUTSIDE TARGET / OUTSIDE FAILURE | Per-guard result |
| Limitation severity | qualitative | L1–L10 (documented limitations) | Methodology scope |
| Framework observations | qualitative | Observations in MAP-STF-FM-001 §7 | Interpretation of the results |

**The three are related but distinct.** A guard status (PASS/OUTSIDE FAILURE) is a per-guard result; a limitation (L1–L10) is a methodology property; a framework observation is an interpretation. **A reader who confuses them may think "AP-03 OUTSIDE FAILURE" means the methodology failed** — it does not; it means the young-portfolio effect is visible in the EAD ratio, and the limitation is documented.

---

## 6. Automated register guards — design

The Fannie Mae DCR uses three register guards, each with a **two-tier structure** (target band vs failure band):

| Guard | Triggers when | Action | Source |
|---|---|---|---|
| **AP-02** | `avg_lgd` outside 40–60% | Log WARNING; if outside 30–70%, log ERROR | DCR §3 |
| **AP-03** | `ead_ratio` outside 60–75% | Log WARNING; if outside 50–85%, log ERROR | DCR §3 |
| **AP-04** | `ecl_ratio` outside 1–3% | Log WARNING; if outside 0.5–5%, log ERROR | DCR §3 |

**The two-tier structure is the DCR's main design choice.** A value outside the target band triggers investigation; a value outside the failure band triggers escalation. **The two tiers distinguish "unusual but acceptable" from "unusual and concerning."**

**On the relationship to Lending Club's register guards.** Lending Club's guards (LC-G1 through LC-G6) fire on model-property thresholds (leakage, fairness, calibration, KS, SHAP, split). Fannie Mae's guards (AP-02 through AP-04) fire on parameter-property thresholds (LGD band, EAD ratio, ECL ratio). **The two numbering schemes are unrelated** — AP-02 in Fannie Mae is not the same as AP-02 in Lending Club, and the two projects' guards fire on different thresholds. A reader comparing the two projects' guards should not assume correspondence.

**The critical point:** register guards fire based on thresholds, not on analyst judgment. The DCR is what makes them automated — a threshold change is a DCR change, not a code change. **A guard that fires without a corresponding DCR entry is a code bug; a DCR entry that does not fire is a data finding.**

---

## 7. Change control and versioning

### 7.1 — Change control rules

| Change type | Version bump | Requires | Example |
|---|---|---|---|
| Parameter bound change (e.g., PD cap 0.20 → 0.25) | **MAJOR** | Methodology Owner approval; full re-run | Changing DCR §3.1 PD cap |
| Register guard threshold change | **MAJOR** | Methodology Owner approval; re-run | Changing AP-03 failure band |
| Scenario weight change | **MAJOR** | Methodology Owner approval; re-run | Changing baseline from 45% to 50% |
| Adding a new guard | **MINOR** | Validator approval; documented | Adding AP-05 for a new metric |
| Clarifying wording | **PATCH** | No approval; documented | Fixing a typo in DCR §3 |

**The rule:** any change that affects a parameter or a register guard's behavior is a **MAJOR** version bump. The Fannie Mae DCR is more sensitive to version bumps than Lending Club's — **because the parameters are frozen in code**, changing them requires a full pipeline re-run.

### 7.2 — Version history

| Version | Date | Change | Type |
|---|---|---|---|
| 1.3 | (frozen at NB03 v1.3) | — | — |
| 1.4 | (this revision) | Structural reorganization: catalogue/results separation, results against criteria in §4, verdict record in §5, LC/AP naming distinction added | MINOR |

**On the v1.3 → v1.4 version bump.** The change is **structural**: §3 is now criterion catalogue only, §4 is results against criteria, §5 is the verdict record, and the LC/AP naming distinction is explicit. **No threshold values changed** — a reader can compare v1.3 and v1.4 and find the same PD cap, LGD bands, AP-02/03/04 thresholds, scenario weights, and reproducibility criteria. The version bump is MINOR because the criteria are identical to v1.3.

---

## 8. Authority and approval

| Change | Proposer | Approver | Rationale required |
|---|---|---|---|
| PD/LGD bounds | IFRS 9 Modeler | Methodology Owner | Yes |
| Register guard thresholds | Modeler | Methodology Owner | Yes |
| Scenario weights | Modeler | Methodology Owner | Yes |
| New register guard | Validator | Methodology Owner | Yes |
| New parameter | Modeler | Methodology Owner | Yes |

**The Methodology Owner is the sole authority for DCR changes.** This mirrors Lending Club's MRC Chair role — the separation of duties is the same.

**The authority chain:**

IFRS 9 Modeler (proposes) → Validator (reviews) → Methodology Owner (approves)


**Unlike Lending Club, the validator's role in DCR changes is advisory, not approval.** The Methodology Owner has the final authority. This is because the parameters are specified, not learned — there is less need for independent validation of the parameter values themselves. **The validator's role in Fannie Mae is to assess the framework (MAP-STF-FM-001), not to approve DCR threshold changes.**

---

## 9. Open items and dependencies

### 9.1 — Open items

| # | Item | Current treatment | Routing |
|---|---|---|---|
| 1 | **PD cap (L6)** | 20% is a conservative cap that masks tail convexity. A future DCR might either remove the cap (with a tail-risk analysis) or justify 20% explicitly. | **MAJOR** version bump if changed; proposed by Modeler, approved by Methodology Owner; tracking in MAP-RP-001 · FM-R-10 |
| 2 | **SICR thresholds (L2)** | The 2× PD increase, 50-point FICO drop, and 30 DPD backstop are analyst-specified. A future DCR might calibrate them against historical data. | **MAJOR** version bump; tracking in MAP-RP-001 · FM-R-07 |
| 3 | **Scenario weights (L9)** | 45/30/15/10 is a reasonable starting point but not calibrated. A future DCR might calibrate against 2008 actuals. | **MAJOR** version bump; tracking in MAP-RP-001 · FM-R-06 |
| 4 | **AP-03 threshold on young portfolios** | The 50–85% failure band is designed for seasoned portfolios. A future DCR might have a separate band for young portfolios (e.g., 60–90%). | **MINOR** version bump; not in the 6-month roadmap (documented gap) |

**On the routing column.** Each open item is routed to the DCR's own change-control chain (§7, §8), and — where applicable — to a MAP-RP-001 finding. A DCR open item is a *proposed DCR change*, not a *finding*; it does not become a finding unless a run shows a defect. The two-track routing (DCR change control + Remediation Plan) reflects that some open items are proposals and some are remediation work.

### 9.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | FM-MD-001 (methodology), FM-RVP-001 (risk & validation plan) |
| Downstream | NB03 (parameter calibration), NB04 (scenarios), NB05 (compliance) |
| Reproducibility | The DCR version is recorded in `notebook_03_parameters.json` |
| Cross-project | The Lending Club guards (`LC-G1`–`LC-G6`) are unrelated to the Fannie Mae register guards (`AP-02`–`AP-04`); see §6 |

---

*End of document.*

