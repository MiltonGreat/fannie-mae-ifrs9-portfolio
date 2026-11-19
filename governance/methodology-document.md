# Methodology Document (A05)

| Metadata Field | Value |
| :--- | :--- |
| **Document ID** | `FM-MD-001` |
| **Version** | `0.2.0-draft` (Forward-Looking Planning Draft) |
| **Run ID / Timestamp** | `YYYYMMDD_HHMMSS` *(literal placeholder; A05 is a planning document and is not bound to a run. Record a run ID only when this document is reviewed against a specific execution artifact.)* |
| **Author / Owner** | `[ANALYST: name, role]` |
| **Project Scope** | IFRS 9 Expected Credit Loss (ECL) Methodology Demonstration (Public Data) |
| **Artifact Type** | Planning document — records the methodology to be built, not a run record |
| **Regulatory Alignment References** | IFRS 9, OSFI E-23, SR 26-2, ECOA *(illustrative/defensibility alignment; not a compliance certification)* |
| **Cross-Referenced Artifacts** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03; aliased as `FM-DS-001`), `FM-FDL-001` (A04), `FM-RVP-001` (A06), `FM-DCR-001` (A07), `FM-CFR-001`, `MAP-STF-FM-001` (framework document), A11 (Results) |

> **Governance & Scope Notice:** This document is a forward-looking planning record of the methodology the IFRS 9 ECL pipeline is designed to implement: how PD, LGD, and EAD are constructed; how loans are staged; how ECL is computed per scenario; and how parameters are bounded. It records the methodology to be built, not the measured values a run produces — measured values belong to the execution artifacts (A11 / DCR §4). It is not a validation opinion, not a production model specification, and not an independent validation report.

> **Status-line convention:** As in A04, each substantive section ends with a status line in one of three states:
> - `Decided` — the methodology decision is made and recorded here.
> - `Deferred to [artifact/notebook] — owner: [role]` — the decision is intentionally not made here; it will be made in the named artifact by the named owner.
> - `Open — decision needed by [milestone]` — no decision exists yet and one is required by a stated milestone.

> **Relationship to other artifacts:** This document is the source the FDL, DCR, and CFR cite when they say "see the MD for the mechanism." The FDL cites the MD for the CCF assertions; the DCR cites the MD for the staging mechanism; the CFR cites the MD for the fairness scoping and limitation mechanisms. This is where those mechanisms are described in one place.

---

## 1. Purpose and scope

This document describes the methodology the Fannie Mae IFRS 9 ECL pipeline is designed to implement: how PD, LGD, and EAD are constructed; how loans are staged; how the ECL is computed per scenario; and how the parameters are bounded.

This is not a validation opinion. The methodology has not been independently validated — the Fannie Mae project is a framework, not a validated model. This document records what the methodology is. The framework document (`MAP-STF-FM-001`) interprets what the methodology's results mean. The methodology's limitations are documented in `FM-CFR-001`.

**Status:** `Decided`

---

## 2. Methodology overview

The pipeline implements IFRS 9's three-component ECL formula:

ECL = PD × LGD × EAD × discount_factor


Where:
- **PD** is a per-loan estimate constructed from FICO, LTV, DTI, and AGE, scaled to `[0.01, 0.20]`.
- **LGD** is a per-loan estimate constructed from a base 45% plus age, LTV, and property-type adjustments, clipped to `[0.10, 0.90]`.
- **EAD** is `ORG_UPB` adjusted for prepayment and curtailment, with `CCF = 0` for term mortgages.
- **discount_factor** is `1 / (1 + 0.035)`, using an industry-average EIR of 3.5%.

The three components are computed per loan, aggregated per stage, and probability-weighted across four scenarios. The measured weighted ECL per loan and portfolio ECL are recorded in **A11 (Results)** and **DCR §4**; they are not restated here.

The methodology is a demonstration, not a production implementation. The parameters are analyst-specified, not fitted. The framework document is explicit about this. The methodology document records what the parameters are; the framework document records what they mean.

**Status:** `Decided`

---

## 3. PD construction

### 3.1 — The `pd_estimate` formula

The per-loan PD is constructed by a specified formula, not fitted by a model:

risk_score = (1 - (FICO_BOR - 620) / 212) × 0.4

    (ORG_LTV / 100) × 0.3

    (DTI / 50) × 0.2

    (AGE / 36) × 0.1

risk_score = risk_score / max(risk_score)

pd_estimate = clip(0.01 + 0.09 × risk_score, 0.01, 0.20)


**On the coefficients.** The weights (0.4, 0.3, 0.2, 0.1) are analyst-specified based on credit-risk theory: FICO is the strongest single predictor, LTV is second, DTI and AGE are weaker. The scaling bounds (0.01, 0.20) are the PD floor and cap. These are not fitted coefficients — a production model would estimate them via logistic regression on the observed default target.

**Where the transform is owned.** Per the analyst's decision in A04, per-feature transforms (including the PD construction) are owned by **A02 §3.3 (Constructed Parameters)** and the NB03 specification. This document describes what the formula does; A02 §3.3 is the authoritative specification.

**Status:** `Decided` for the formula as recorded here; `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` for the authoritative transform specification.

### 3.2 — Lifetime PD

The lifetime PD for Stage 2 and Stage 3 loans is computed from the unconditional PD by:

lifetime_pd = 1 - (1 - unconditional_pd)^30


Clipped to `[0.01%, 20%]`. When the cap fires, the pre-cap value is disclosed as `L6` in `FM-CFR-001`. The measured pre-cap and post-cap lifetime PD are recorded in **A11 / DCR §4**, not here.

**Status:** `Decided`

### 3.3 — Stage distribution

Loans are classified into IFRS 9 Stages 1, 2, and 3:
- **Stage 3:** `default == 1` (the `delinquency_state >= 3` condition is equivalent, because `delinquency_state = default × 5` — the `C6` limitation).
- **Stage 2:** top-decile of non-Stage-3 loans by `pd_estimate` (SICR-by-construction, not observed credit deterioration — `L2`).
- **Stage 1:** all other loans.

The Stage 2 mechanism is the methodology's most important interpretive point. A reader must not read the Stage 2 fraction as "the share of the portfolio that had a significant increase in credit risk." It means "the top-decile proxy assigns that share of loans to Stage 2." The measured stage distribution is recorded in **A11 / DCR §4**, not here.

**Status:** `Decided` for the mechanism; `Deferred to A11 / DCR §4` for the measured fractions.

---

## 4. LGD and EAD construction

### 4.1 — LGD

Per-loan LGD is:

lgd_final = base_lgd + age_adjustment + ltv_adjustment + property_adjustment


Clipped to `[0.10, 0.90]`.

| Adjustment | Values |
| :--- | :--- |
| Base LGD | 45% |
| Age | +10% (0–12m), +5% (12–24m), 0% (24–36m), −5% (36–60m), −10% (60–120m), −15% (120m+) |
| LTV | −10% (Low), 0% (Medium), +15% (High) |
| Property | SF: −5%, PU: 0%, CO: +10%, MH: +20%, CP: +15% |
| Floor / cap | 10% / 90% |

The values are analyst-specified. Per the analyst's decision in A04, the authoritative transform specification is **A02 §3.3 / NB03**. A production implementation would fit the adjustments against observed recovery data; here they are specified because the goal is methodology demonstration. The measured average LGD is recorded in **A11 / DCR §4**, not here.

**Status:** `Decided` for the construction as recorded here; `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` for the authoritative transform specification; `Deferred to A11 / DCR §4` for the measured average LGD.

### 4.2 — EAD

Per-loan EAD is:

ead_final = ORG_UPB × prepayment_factor × curtailment_factor


Where:
- `prepayment_factor = clip(1 - 0.08 × AGE/12, 0.5, 1.0)`
- `curtailment_factor = clip(1 - 0.02 × AGE/12, 0.7, 1.0)`
- `CCF = 0.0` for term mortgages.

The measured average EAD and EAD ratio are recorded in **A11 / DCR §4**, not here.

**Status:** `Decided` for the construction; `Deferred to A11 / DCR §4` for the measured values.

### 4.3 — The three CCF = 0 assertions

The DCR (§3.1) says CCF is 0 for term mortgages. Any nonzero CCF is a specification violation. The pipeline asserts this at three boundaries:

1. **Config assertion.** `IFRS9Config.product_type == 'term_mortgage'` → `config.ccf` must be 0. Asserted at config initialization.
2. **Computation assertion.** In `EADCalculator.calculate_ead`, after computing `ccf = 0.0`, the code asserts `df_obs['ccf'].eq(0.0).all()`.
3. **Return assertion.** The `ead_estimates` dict records `'ccf_used': 0.0`; the return assertion verifies the value before the dict is returned.

The DCR's §3.1 is the policy; A02 §3.3 is the transform specification; this section is the mechanism. The rationale for the three-assertion implementation is planned in §7.4.

**Status:** `Decided` for the mechanism; `Open — decision needed by [ANALYST: milestone]` for the three-assertion rationale (see §7.4).

---

## 5. ECL computation

### 5.1 — Per-stage ECL

ECL is computed per stage, with a stage PD multiplier:

| Stage | PD multiplier |
| :--- | :--- |
| Stage 1 | 1.0× |
| Stage 2 | 2.0× |
| Stage 3 | 3.0× |

Multipliers are applied to the unconditional PD, clipped at 20%.

**On the multipliers.** The 1.0×/2.0×/3.0× multipliers are analyst-specified. They are a simplified proxy for the lifetime-PD distinction that a production implementation would compute directly. The methodology does not implement a true lifetime PD for Stages 2 and 3 — it uses multipliers. This is documented as `L2` in `FM-CFR-001`.

**Status:** `Decided`

### 5.2 — Scenario-weighted ECL

Four scenarios, each with a PD multiplier and an LGD multiplier:

| Scenario | Weight | PD multiplier | LGD multiplier |
| :--- | :--- | :--- | :--- |
| Baseline | 45% | 1.0× | 1.0× |
| Adverse | 30% | 1.8× | 1.2× |
| Severely Adverse | 15% | 3.0× | 1.5× |
| Tail Risk | 10% | 4.5× | 2.0× |

Per-scenario ECL is computed, then weighted by scenario weight, then summed. The weights sum to 1.0 — a hard requirement checked at runtime.

The measured weighted ECL per loan and the severity-check ratio are recorded in **A11 / DCR §4**, not here.

**On the severity check.** The severely adverse ECL rate vs. the 2008–2010 historical average is compared against the DCR's 1.5× target. The measured ratio is recorded in A11 / DCR §4. This is not framed here as a pass/fail; the framework document (`MAP-STF-FM-001`) records the observation.

**Status:** `Decided` for the scenario design; `Deferred to A11 / DCR §4` for the measured weighted ECL and severity ratio.

### 5.3 — Discount factor

discount_factor = 1 / (1 + 0.035)


The 3.5% EIR is an industry average (`L5`).

**Status:** `Decided`

---

## 6. Fairness testing

### 6.1 — What is tested

The pipeline runs fairness testing on the geographic proxies it has: state, MSA, ZIP. The results are reported with an explicit `UNRELIABLE` flag, because the 4/5ths rule at cardinality 52 / 328 / 638 produces degenerate results (many groups with zero defaults → DI = 0.00%, which is not evidence of discrimination).

**What is not tested:** race, ethnicity, sex, or any other direct protected attribute. The dataset does not contain them. The framework document's "Fairness testing" section is the authoritative statement of this limitation.

**Status:** `Decided`

### 6.2 — What the scoping means

The fairness testing is incomplete, not failed. The `UNRELIABLE` flag is the notebook's mechanism for saying "this test is degenerate; do not read the DI ratio as a finding." The distinction matters: `UNRELIABLE` means the test cannot tell you; `Non-Compliant` would mean the test was performed and failed.

The analyst, writing the Validation Opinion, can honestly write "Not Assessed" rather than "Non-Compliant" — because the notebook's flag is the evidence for that status. This is documented as `L7` in `FM-CFR-001`.

**Status:** `Decided`

---

## 7. Bounds and their rationale

### 7.1 — The four bound families

| Family | Bounds | Source |
| :--- | :--- | :--- |
| PD | Floor 0.01%, cap 20% | DCR §3.1 |
| LGD | Floor 10%, cap 90% | DCR §3.1 |
| EAD | CCF = 0 for term mortgages | DCR §3.1 |
| Scenario | Weights sum to 1.0 | DCR §3.4 |

**Status:** `Decided`

### 7.2 — The PD cap

The 20% PD cap masks tail convexity (`L6`). It is a stability measure, not a calibration artifact. Changing the cap is a MAJOR DCR change (see DCR §7.1).

**Planned rationale (to be written):** Why 20% and not 15% or 25%. The DCR records 20% as a conservative cap; the specific choice of 20% is to be justified here, with a reference to any analysis that supported it. If no analysis supported it, this section will state that explicitly — a bound without a rationale is a bound a future reviewer will question.

**Status:** `Open — decision needed by [ANALYST: milestone]` (rationale to be written; bound value itself is `Decided` at 20%)

### 7.3 — The LGD floor and cap

The 10% LGD floor and 90% LGD cap are safety bounds. The floor prevents a zero-loss scenario for a performing loan; the cap prevents a total-loss scenario for a loan with any expected recovery. The 90% cap is especially important — it distinguishes "the loan's expected loss is nearly total" from "the loan's expected loss is total."

**Planned rationale (to be written):** Why 10% and 90% and not 5% and 95%. If there is a rationale, this section will state it; if not, this section will state that the bounds are conventional.

**Status:** `Open — decision needed by [ANALYST: milestone]` (rationale to be written; bound values themselves are `Decided` at 10%/90%)

### 7.4 — The CCF = 0 rule

Term mortgages have no undrawn commitment, so CCF is 0. The rule is asserted three times (see §4.3). Changing it is a MAJOR DCR change.

**Planned rationale (to be written):** Why the three-assertion implementation. The DCR records CCF = 0 as the rule; "assert at three boundaries" is an implementation decision. This section will state what failure mode the third assertion catches that the first two do not — whether the answer is "defense in depth" or "the third assertion was added after a bug." The latter is more instructive if it is the case.

**Status:** `Open — decision needed by [ANALYST: milestone]` (rationale to be written; rule itself is `Decided`)

---

## 8. Limitations and their mechanism

Every limitation in `FM-CFR-001` is a property of the methodology, not a defect in its implementation. This section records the mechanism for each:

| Limitation | Mechanism |
| :--- | :--- |
| `L1` — PD-LGD correlation is constructed | LGD is a function of PD via the shared age and LTV inputs. The "correlation" is definitional, not empirical. |
| `L2` — SICR uses top-decile proxy | The Stage 2 population is the top decile by `pd_estimate`, not an observed credit deterioration. |
| `L3` — LGD is a function of PD | Same mechanism as `L1`. |
| `L4` — Prepayment/curtailment are assumptions | 8% and 2% are historical averages; they are not calibrated to the portfolio. |
| `L5` — EIR is industry average | 3.5% is a conventional discount rate; the portfolio's actual EIR is not used. |
| `L6` — PD cap masks tail convexity | The 20% cap clips the lifetime PD and the tail-scenario PD. |
| `L7` — Fairness testing incomplete | No direct protected attributes in the dataset. |
| `L8` — U.S. data only | The dataset is Fannie Mae; no Canadian portfolio validation. |
| `L9` — Scenario weights and multipliers are assumptions | The 45/30/15/10 weights and the 1.0–4.5× multipliers are analyst-specified. |
| `L10` — Capital coverage is a diagnostic | The ECL/CET1 ratio is a scale artifact; it is not a capital adequacy finding. |

The mechanism column is this document's contribution to the CFR. The CFR records what the limitation is; this document records how the methodology produces it.

**Status:** `Decided`

---

## 9. Deferral and open-item register (A05 scope)

Every `Deferred` and `Open` status line in §§1–8 has a row here.

| # | Item | Deferred to | Owner | Trigger/milestone |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Authoritative PD transform specification | A02 §3.3 / NB03 | `[ANALYST: role]` | NB03 specification |
| 2 | Authoritative LGD transform specification | A02 §3.3 / NB03 | `[ANALYST: role]` | NB03 specification |
| 3 | Measured PD / lifetime PD values | A11 / DCR §4 | `[ANALYST: role]` | A11 finalization |
| 4 | Measured stage distribution | A11 / DCR §4 | `[ANALYST: role]` | A11 finalization |
| 5 | Measured average LGD | A11 / DCR §4 | `[ANALYST: role]` | A11 finalization |
| 6 | Measured average EAD and EAD ratio | A11 / DCR §4 | `[ANALYST: role]` | A11 finalization |
| 7 | Measured weighted ECL and severity ratio | A11 / DCR §4 | `[ANALYST: role]` | A11 finalization |
| 8 | Rationale for the 20% PD cap (§7.2) | A05 §7.2 | `[ANALYST: role]` | Before DCR MAJOR bump |
| 9 | Rationale for the 10%/90% LGD bounds (§7.3) | A05 §7.3 | `[ANALYST: role]` | Before DCR MAJOR bump |
| 10 | Rationale for the three CCF assertions (§7.4) | A05 §7.4 | `[ANALYST: role]` | Before DCR MAJOR bump |

---

## 10. Document control

| Field | Value |
| :--- | :--- |
| **Document ID** | `FM-MD-001` |
| **Version** | `0.2.0-draft` |
| **Status** | Draft — planning document; pending analyst completion of the §7.2–§7.4 rationales and the §9 deferral items |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | `0.1` |
| **Related documents** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03; aliased as `FM-DS-001`), `FM-FDL-001` (A04), `FM-RVP-001` (A06), `FM-DCR-001` (A07), `FM-CFR-001`, `MAP-STF-FM-001` (framework document), A11 (Results) |
| **Next review** | At the next MAJOR version bump of `FM-DCR-001`, or upon completion of the §7.2–§7.4 rationales |
