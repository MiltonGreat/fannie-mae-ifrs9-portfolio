# Analytical Specification — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-AS-001  
**Version / Status:** 0.1 / Draft — retrospective  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Analytical Specification (AS)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

> **Retrospective framing.** This document was reconstructed from the FM pipeline's implementation, not written before it. The specification the pipeline implements is documented here; a specification written before the analysis would have been the document that the pipeline was required to conform to. **Where this document and the FDL/DCR/CFR disagree, the FDL/DCR/CFR take precedence** — they were revised against the notebook output, while this document is a reconstruction.

---

## 1. Purpose and scope

This AS records the analytical specification the Fannie Mae IFRS 9 ECL pipeline implements: the research questions, the hypotheses, the variables, and the success criteria.

**This is not a validation opinion.** The specification records what the pipeline was supposed to do; it does not record whether the pipeline did it well. The framework document (MAP-STF-FM-001) is where the *results* are assessed; this document is where the *specification* is recorded.

**The retrospective framing matters.** A pre-analysis AS would have constrained the subsequent work; this AS records what the work turned out to be. The two are different documents, and the difference is documented in the "Retrospective framing" note above.

---

## 2. Research questions

The pipeline answers four research questions:

| # | Research question | What the answer would mean |
|---|---|---|
| RQ1 | Does the pipeline implement the IFRS 9 ECL arithmetic (PD × LGD × EAD × discount) correctly? | Yes → the methodology is demonstrable; No → the pipeline has a defect |
| RQ2 | Does the pipeline's staging approximate IFRS 9 Stages 1, 2, 3? | Yes → the methodology's staging is a valid proxy; No → the methodology's staging is invalid |
| RQ3 | Does the pipeline's probability-weighted ECL produce a defensible estimate under the four scenarios? | Yes → the scenario design is defensible; No → the scenarios or the weighting need revision |
| RQ4 | Do the register guards (AP-02, AP-03, AP-04) fire as designed? | Yes → the guards are calibrated; No → a guard needs threshold calibration |

**What the pipeline answers vs. what it does not answer:**
- It answers RQ1–RQ4 for the Fannie Mae sample and the analyst-specified parameters.
- It does not answer whether the parameters are correct (that is the DCR's question, and the DCR records them as analyst-specified, not calibrated).
- It does not answer whether the resulting ECL is a production estimate (that is the framework document's question, and the framework document says no).

`[ANALYST: insert the analyst's framing of the research questions. The four RQs above are the questions the pipeline answers. The analyst's framing is: why these four and not others? What question would a reader expect that the pipeline does not answer? If the answer is "whether the ECL is correct for a real portfolio," state it — it is the most important question the pipeline does not answer.]`

---

## 3. Hypotheses

The pipeline tests the following hypotheses:

| # | Hypothesis | Test | Acceptance criterion |
|---|---|---|---|
| H1 | The `pd_estimate` formula produces a PD range consistent with the DCR's bounds | Compare `pd_estimate` values to [0.01, 0.20] | Values within bounds |
| H2 | The `lgd_final` values are in the DCR's 40–60% target band | Compute `lgd_final.mean()` | AP-02 PASS |
| H3 | The `ead_final` values produce an EAD ratio in the DCR's 50–85% failure band | Compute EAD ratio | AP-03 PASS or documented exception |
| H4 | The `ecl_final` (weighted ECL) produces an ECL ratio in the DCR's 1–3% target band | Compute ECL ratio | AP-04 PASS |
| H5 | The staging distribution produces non-degenerate fractions (all three stages non-empty) | Compute stage distribution | Stage 1 > 0, Stage 2 > 0, Stage 3 > 0 |
| H6 | The scenario multipliers produce a weighted ECL consistent with the severity target | Compute severity ratio | Severity ratio ≥ 1.5× (per DCR) |

**In this run:**

| Hypothesis | Result |
|---|---|
| H1 | **Accepted** — `pd_estimate` values within bounds |
| H2 | **Accepted** — LGD 44.28%, AP-02 PASS |
| H3 | **Rejected as expected** — EAD ratio 86.95%, AP-03 fires (documented exception) |
| H4 | **Accepted** — ECL ratio 1.09%, AP-04 PASS |
| H5 | **Accepted** — 89.1% / 9.9% / 1.0% |
| H6 | **Rejected** — severity ratio 1.27×, below the 1.5× target |

**On the two rejections.** H3's rejection is expected — the young portfolio makes the EAD ratio high, and the DCR documents this as a designed outcome. H6's rejection is **not** expected — it is the DCR's AP-05 finding (severity is below target), and it is the framework document's most important substantive result.

`[ANALYST: insert the analyst's framing of the two rejections. H3's rejection is documented; H6's rejection is the substantive finding. The framing is: which of the two matters more for a reader, and why? The answer is probably H6 — a young-portfolio effect is a data property, while a severity shortfall is a methodology property. If the analyst's framing is different, say so.]`

---

## 4. Variables

### 4.1 — Target

| Variable | Role | Definition |
|---|---|---|
| `default` | Target | Binary; 1 = bad status, 0 = good status |

### 4.2 — Inputs

| Variable | Role | Source |
|---|---|---|
| `FICO_BOR` | PD input; LGD covariate | Origination |
| `ORG_LTV` | PD input; LGD covariate | Origination |
| `DTI` | PD input | Origination |
| `AGE` | PD input; LGD covariate | Current state |
| `ORG_UPB` | EAD input | Origination |
| `ORG_CLTV` | LGD covariate | Origination |
| `PROP_TYPE` | LGD covariate | Origination |
| `OCCU_STAT` | Possible signal | Origination |

### 4.3 — Constructed parameters

| Variable | Role | Construction |
|---|---|---|
| `pd_estimate` | ECL input | Weighted FICO/LTV/DTI/AGE; scaled to [0.01, 0.10] in this run |
| `lgd_final` | ECL input | Base 45% + age/LTV/property adjustments; clipped [0.10, 0.90] |
| `ead_final` | ECL input | `ORG_UPB` × prepayment × curtailment; CCF = 0 |

### 4.4 — Excluded variables

The four target transforms (`delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag`) are excluded from the feature matrix. The full exclusion list is in FM-FDL-001 §7.

`[ANALYST: insert the analyst's framing of the variable set. The variable set is small — eight inputs for the ECL. Why only eight? The FDL says "the current pipeline's feature set is deliberately small because the goal is to demonstrate the ECL methodology, not to build the strongest possible default predictor." The analyst's framing is whether that choice is right: is eight the right number for a methodology demonstration? Would a larger set be more convincing or less focused?]`

---

## 5. Success criteria

The pipeline's success criteria are the criteria the DCR (FM-DCR-001) defines:

| Criterion | Threshold | Source |
|---|---|---|
| AP-02 (LGD band) | 40%–60% target; 30%–70% failure | DCR §3.3 |
| AP-03 (EAD ratio) | 60%–75% target; 50%–85% failure | DCR §3.3 |
| AP-04 (ECL ratio) | 1%–3% target; 0.5%–5% failure | DCR §3.3 |
| AP-05 (severity) | ≥ 1.5× historical average | DCR §3.4 |

**The pipeline's success is not "all criteria pass."** The pipeline's success is that the criteria **fire as designed** — including the criteria that fire because the data has a known property (AP-03) and the criteria that fire because the methodology has a known limitation (AP-05).

**In this run:** AP-02 PASS, AP-03 OUTSIDE FAILURE (expected), AP-04 PASS, AP-05 OUTSIDE TARGET (severity ratio 1.27× < 1.5×).

**The AP-05 finding is the methodology's most important result.** It says the assumed scenario multipliers produce a scenario the register considers insufficiently severe. This is not a failure of the pipeline — it is a finding about the *scenario design*, documented as L9 in FM-CFR-001.

`[ANALYST: insert the analyst's framing of "success" for the specification. The criteria are the DCR's; the "fire as designed" framing is this document's. The analyst's framing is: which criterion's result is the most important for a reader to understand? The answer is probably AP-05 — a guard firing as designed is fine; a guard firing because the methodology is short of target is the finding.]`

---

## 6. Assumptions

The AS records the assumptions the specification rests on. They are:

| # | Assumption | Source |
|---|---|---|
| 1 | Fannie Mae public data is a valid proxy for a mortgage portfolio | FM-PDR-001 §3 |
| 2 | 16+ days past due is the right target definition for a collections-prioritization model | FM-PDR-001 §4 |
| 3 | PD/LGD/EAD can be constructed from the available origination and current-state fields | FM-FDL-001 §5 |
| 4 | The scenario multipliers (1.0–4.5×) produce a defensible stress scenario | FM-DCR-001 §3.4 |
| 5 | The four scenarios (45/30/15/10) are a defensible probability weighting | FM-DCR-001 §3.4 |
| 6 | The register guards (AP-02 through AP-05) are calibrated to the methodology's intended scope | FM-DCR-001 §3.3 |

**The assumptions are analyst judgments, not methodological conclusions.** Each one could be challenged; each challenge would be a valid input to a validation opinion (which the framework does not have). **The AS records the assumptions; it does not defend them.** The defense is in the framework document and, where applicable, the analyst's approval record.

`[ANALYST: insert the analyst's framing of the assumptions. The six assumptions are the specification's premises. The analyst's framing is: which of the six is the most consequential? Which would a reader most want challenged? The answer is probably assumption 4 (the scenario multipliers) — it is the assumption with the largest effect on the headline numbers, and the one most likely to be questioned by a validator.]`

---

## 7. Open items and dependencies

### 7.1 — Open items

| # | Item | Current state | Routing |
|---|---|---|---|
| 1 | The variable set is small (8 inputs for the ECL) | Deliberate for a methodology demonstration | Not in the current roadmap |
| 2 | The assumptions are analyst judgments, not conclusions | Documented in §6 | Framework document's "What this does NOT demonstrate" |
| 3 | The retrospective framing | This document was reconstructed, not pre-written | Stated in the document header |
| 4 | The AP-05 severity finding | Recorded as L9; AP-05 fires | FM-CFR-001 §7.1; MAP-RP-001 FM-R-06 |

**The retrospective framing is the AS's most important open item.** A pre-analysis AS would have been the document that constrained the subsequent work; this AS records what the work turned out to be. **A future version could reconstruct a pre-analysis AS from this one**, or a future validation could treat this document as the specification of record. The choice is the analyst's.

### 7.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | FM-PDR-001 (problem definition), FM-DS-001 (data specification) |
| Downstream | FM-FDL-001 (feature decisions), FM-MD-001 (methodology), FM-DCR-001 (decision criteria), FM-CFR-001 (findings), FM-RVP-001 (risk & validation plan) |
| Cross-project | The LC project's AS is a separate document with a different structure; the two ASs are not directly comparable |

---

## 8. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-AS-001 |
| **Version** | 0.1 |
| **Status** | Draft — retrospective; pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | At the next MAJOR version bump of the framework |

---

## 9. What this document contains, and what it does not

### What it contains (from the notebook output and the FM documents)

- The four research questions.
- The six hypotheses and their test results.
- The variables (target, inputs, constructed parameters, excluded).
- The success criteria (the DCR's guard bands).
- The assumptions (from the FDL, DCR, and PDR).
- The open items and dependencies.
- The retrospective framing note.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§2** — the analyst's framing of the research questions.
- **§3** — the analyst's framing of the two rejected hypotheses.
- **§4** — the analyst's framing of the variable set.
- **§5** — the analyst's framing of "success."
- **§6** — the analyst's framing of the assumptions.
- **§8** — the document owner's name and role.

**The document's structure is retrospective.** The factual content (research questions, hypotheses, variables, criteria, assumptions) is sourced from the notebook output and the FM documents. The analytical content (why these questions, which hypotheses matter, which assumptions are most consequential) is the analyst's contribution.
