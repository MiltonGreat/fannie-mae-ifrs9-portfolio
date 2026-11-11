# Methodology Document — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-MD-001  
**Version / Status:** 0.1 / Draft  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Methodology Document (MD)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

## 1. Purpose and scope

This document describes the methodology the Fannie Mae IFRS 9 ECL pipeline implements: how PD, LGD, and EAD are constructed; how loans are staged; how the ECL is computed per scenario; and how the parameters are bounded.

**This is not a validation opinion.** The methodology has not been independently validated — the Fannie Mae project is a framework, not a validated model. This document records what the methodology is. The framework document (MAP-STF-FM-001) interprets what the methodology's results mean. The methodology's limitations are documented in FM-CFR-001.

**This document is the source the FDL, DCR, and CFR cite when they say "see the MD for the mechanism."** The FDL's §4.2 cites the MD for the CCF assertions; the DCR's §3.2 cites the MD for the staging mechanism; the CFR's §6.1 cites the MD for the fairness scoping. This document is where those mechanisms are described in one place.

---

## 2. Methodology overview

The pipeline implements IFRS 9's three-component ECL formula:

```
ECL = PD × LGD × EAD × discount_factor
```

Where:

- **PD** (Probability of Default) is a per-loan estimate constructed from FICO, LTV, DTI, and AGE, scaled to [0.01, 0.20].
- **LGD** (Loss Given Default) is a per-loan estimate constructed from a base 45% plus age, LTV, and property-type adjustments, clipped to [0.10, 0.90].
- **EAD** (Exposure at Default) is `ORG_UPB` adjusted for prepayment and curtailment, with CCF = 0 for term mortgages.
- **discount_factor** is `1 / (1 + 0.035)`, using an industry-average EIR of 3.5%.

The three components are computed per loan, aggregated per stage, and probability-weighted across four scenarios. The framework document reports the weighted ECL per loan and the portfolio ECL.

**The methodology is a demonstration, not a production implementation.** The parameters are analyst-specified, not fitted. The framework document is explicit about this (see its "What this framework does NOT demonstrate" section). The methodology document records *what the parameters are*; the framework document records *what they mean*.

---

## 3. PD construction

### 3.1 — The `pd_estimate` formula

The per-loan PD is constructed by a specified formula, not fitted by a model:

```
risk_score = (1 - (FICO_BOR - 620) / 212) × 0.4
             + (ORG_LTV / 100) × 0.3
             + (DTI / 50) × 0.2
             + (AGE / 36) × 0.1

risk_score = risk_score / max(risk_score)

pd_estimate = clip(0.01 + 0.09 × risk_score, 0.01, 0.20)
```

**On the coefficients.** The weights (0.4, 0.3, 0.2, 0.1) are analyst-specified based on credit-risk theory: FICO is the strongest single predictor, LTV is second, DTI and AGE are weaker. The scaling bounds (0.01, 0.20) are the PD floor and cap. **These are not fitted coefficients** — a production model would estimate them via logistic regression on the observed `default` target.

**The FDL is where the formula lives.** FM-FDL-001 §5.2 is the authoritative source for the formula. This document describes what the formula does; the FDL records the formula itself. If the two ever disagree, the FDL is correct — the formula is a feature decision, and the FDL is the feature decision log.

### 3.2 — Lifetime PD

The lifetime PD for Stage 2 and Stage 3 loans is computed from the unconditional PD by:

```
lifetime_pd = 1 - (1 - unconditional_pd)^30
```

Clipped to [0.01%, 20%].

**In this run:** `1 - (1 - 0.0095)^30 ≈ 26.1%`, clipped to 20%. **The cap fires.** The pre-cap value is disclosed as L6 in FM-CFR-001.

### 3.3 — Stage distribution

Loans are classified into IFRS 9 Stages 1, 2, and 3:

- **Stage 3:** `default == 1` (the `delinquency_state >= 3` condition is equivalent, because `delinquency_state = default × 5` — the C6 limitation).
- **Stage 2:** top-decile of non-Stage-3 loans by `pd_estimate` (**SICR-by-construction**, not observed credit deterioration — L2).
- **Stage 1:** all other loans.

**In this run:** Stage 1 = 89.1%, Stage 2 = 9.9%, Stage 3 = 1.0%.

**The Stage 2 mechanism is the methodology's most important interpretive point.** A reader must not read "Stage 2 = 9.9%" as "9.9% of the portfolio had a significant increase in credit risk." It means "the top-decile proxy assigns 9.9% of loans to Stage 2." The framework document's §3 carries the same language.

---

## 4. LGD and EAD construction

### 4.1 — LGD

Per-loan LGD is:

```
lgd_final = base_lgd + age_adjustment + ltv_adjustment + property_adjustment
```

Clipped to [0.10, 0.90].

| Adjustment | Values |
|---|---|
| Base LGD | 45% |
| Age | +10% (0–12m), +5% (12–24m), 0% (24–36m), −5% (36–60m), −10% (60–120m), −15% (120m+) |
| LTV | −10% (Low), 0% (Medium), +15% (High) |
| Property | SF: −5%, PU: 0%, CO: +10%, MH: +20%, CP: +15% |
| Floor / cap | 10% / 90% |

**In this run:** average LGD = 44.28% (inside the 40–60% benchmark band; AP-02 PASS).

**The values are analyst-specified.** The FDL's §5.3 is the authoritative source. A production implementation would fit the adjustments against observed recovery data; here they are specified because the goal is methodology demonstration.

### 4.2 — EAD

Per-loan EAD is:

```
ead_final = ORG_UPB × prepayment_factor × curtailment_factor
```

Where:

- `prepayment_factor = clip(1 - 0.08 × AGE/12, 0.5, 1.0)`
- `curtailment_factor = clip(1 - 0.02 × AGE/12, 0.7, 1.0)`
- **CCF = 0.0** for term mortgages.

**In this run:** average EAD = $204,745; EAD ratio = 86.95% (outside the 50–85% failure band; AP-03 fires; expected on a young portfolio).

### 4.3 — The three CCF = 0 assertions

The DCR (§3.1) says CCF is 0 for term mortgages. **Any nonzero CCF is a specification violation.** The pipeline asserts this at three boundaries:

1. **Config assertion.** `IFRS9Config.product_type == 'term_mortgage'` → `config.ccf` must be 0. Asserted at config initialization.
2. **Computation assertion.** In `EADCalculator.calculate_ead`, after computing `ccf = 0.0`, the code asserts `df_obs['ccf'].eq(0.0).all()`.
3. **Return assertion.** The `ead_estimates` dict records `'ccf_used': 0.0`; the return assertion verifies the value before the dict is returned.

**The three assertions are the DCR's most important rule at the code level.** A reader who sees "CCF = 0" in the DCR and "CCF = 0" in the FDL should know that the pipeline enforces it three times. The DCR's §3.1 is the policy; the FDL's §5.1 is the feature decision; this section is the mechanism.

---

## 5. ECL computation

### 5.1 — Per-stage ECL

ECL is computed per stage, with a stage PD multiplier:

| Stage | PD multiplier |
|---|---|
| Stage 1 | 1.0× |
| Stage 2 | 2.0× |
| Stage 3 | 3.0× |

Multipliers are applied to the unconditional PD, clipped at 20%.

**On the multipliers.** The 1.0×/2.0×/3.0× multipliers are analyst-specified. They are a simplified proxy for the lifetime-PD distinction that a production implementation would compute directly. **The methodology does not implement a true lifetime PD for Stage 2 and 3** — it uses multipliers. This is documented as L2 in FM-CFR-001.

### 5.2 — Scenario-weighted ECL

Four scenarios, each with a PD multiplier and an LGD multiplier:

| Scenario | Weight | PD multiplier | LGD multiplier |
|---|---|---|---|
| Baseline | 45% | 1.0× | 1.0× |
| Adverse | 30% | 1.8× | 1.2× |
| Severely Adverse | 15% | 3.0× | 1.5× |
| Tail Risk | 10% | 4.5× | 2.0× |

Per-scenario ECL is computed, then weighted by scenario weight, then summed. **The weights sum to 1.0** — a hard requirement checked at runtime.

**In this run:** weighted ECL per loan = $2,231.16.

**On the severity check.** The severely adverse ECL rate vs. the 2008–2010 historical average is ~1.27×, **below the DCR's 1.5× target**. This is not a methodology failure — it is a documented finding (AP-05, IT-05) that the assumed multipliers produce a scenario the register considers insufficiently severe.

### 5.3 — Discount factor

```
discount_factor = 1 / (1 + 0.035)
```

The 3.5% EIR is an industry average (L5).

---

## 6. Fairness testing

### 6.1 — What is tested

The pipeline runs fairness testing on the geographic proxies it has: state, MSA, ZIP. The results are reported with an explicit `UNRELIABLE` flag, because the 4/5ths rule at cardinality 52 / 328 / 638 produces degenerate results (many groups with zero defaults → DI = 0.00%, which is not evidence of discrimination).

**What is not tested:** race, ethnicity, sex, or any other direct protected attribute. The dataset does not contain them. The framework document's "Fairness testing" section is the authoritative statement of this limitation.

### 6.2 — What the scoping means

**The fairness testing is incomplete, not failed.** The `UNRELIABLE` flag is the notebook's mechanism for saying "this test is degenerate; do not read the DI ratio as a finding." The distinction matters: `UNRELIABLE` means the test cannot tell you; `Non-Compliant` would mean the test was performed and failed.

**The analyst, writing the Validation Opinion, can honestly write "Not Assessed" rather than "Non-Compliant"** — because the notebook's flag is the evidence for that status. This is documented as L7 in FM-CFR-001.

---

## 7. Bounds and their rationale

### 7.1 — The four bound families

| Family | Bounds | Source |
|---|---|---|
| PD | Floor 0.01%, cap 20% | DCR §3.1 |
| LGD | Floor 10%, cap 90% | DCR §3.1 |
| EAD | CCF = 0 for term mortgages | DCR §3.1 |
| Scenario | Weights sum to 1.0 | DCR §3.4 |

### 7.2 — The PD cap

The 20% PD cap masks tail convexity (L6). It is a stability measure, not a calibration artifact. **Changing the cap is a MAJOR DCR change** (see DCR §7.1).

`[ANALYST: insert the reasoning for why 20% and not 15% or 25%. The DCR says "20% is a conservative cap"; the specific choice of 20% should be justified here, with a reference to any analysis that supported it. If no analysis supported it, state that explicitly — a bound without a rationale is a bound that a future reviewer will question.]`

### 7.3 — The LGD floor and cap

The 10% LGD floor and 90% LGD cap are safety bounds. The floor prevents a zero-loss scenario for a performing loan; the cap prevents a total-loss scenario for a loan with any expected recovery. **The 90% cap is especially important** — it is what distinguishes "the loan's expected loss is nearly total" from "the loan's expected loss is total."

`[ANALYST: insert the reasoning for why 10% and 90% and not 5% and 95%. Same pattern as §7.2 — if there is a rationale, state it; if not, state that the bounds are conventional.]`

### 7.4 — The CCF = 0 rule

Term mortgages have no undrawn commitment, so CCF is 0. **The rule is asserted three times** (see §4.3). Changing it is a MAJOR DCR change.

`[ANALYST: insert the reasoning for the three-assertion implementation. The DCR says CCF=0 is the rule; the "assert at three boundaries" choice is an implementation decision. Why three? What failure mode does the third assertion catch that the first two do not? If the answer is "defense in depth," state that. If the answer is "the third assertion was added after a bug," state that — it is more instructive.]`

---

## 8. Limitations and their mechanism

Every limitation in FM-CFR-001 is a *property of the methodology*, not a defect in its implementation. This section records the mechanism for each:

| Limitation | Mechanism |
|---|---|
| L1 — PD-LGD correlation is constructed | LGD is a function of PD via the shared age and LTV inputs. The "correlation" is definitional, not empirical. |
| L2 — SICR uses top-decile proxy | The Stage 2 population is the top decile by `pd_estimate`, not an observed credit deterioration. |
| L3 — LGD is a function of PD | Same mechanism as L1. |
| L4 — Prepayment/curtailment are assumptions | 8% and 2% are historical averages; they are not calibrated to the portfolio. |
| L5 — EIR is industry average | 3.5% is a conventional discount rate; the portfolio's actual EIR is not used. |
| L6 — PD cap masks tail convexity | The 20% cap clips the lifetime PD and the tail-scenario PD. |
| L7 — Fairness testing incomplete | No direct protected attributes in the dataset. |
| L8 — U.S. data only | The dataset is Fannie Mae; no Canadian portfolio validation. |
| L9 — Scenario weights and multipliers are assumptions | The 45/30/15/10 weights and the 1.0–4.5× multipliers are analyst-specified. |
| L10 — Capital coverage is a diagnostic | The ECL/CET1 ratio is a scale artifact; it is not a capital adequacy finding. |

**The mechanism column is this document's contribution to the CFR.** The CFR records what the limitation is; this document records *how the methodology produces it*. A reader who wants to understand why the limitation exists follows the mechanism column; a reader who wants to know the severity follows the CFR.

---

## 9. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-MD-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | At the next MAJOR version bump of FM-DCR-001 |

---

## 10. What this document contains, and what it does not

### What it contains

- The three-component ECL formula and its four inputs.
- The PD, LGD, and EAD construction, with formulas and bounds.
- The three-assertion CCF mechanism.
- The staging mechanism and the stage PD multipliers.
- The four-scenario probability weighting.
- The fairness scoping.
- The PD cap, LGD bounds, and CCF rationale (with markers).
- The mechanism for each CFR limitation.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§7.2** — the rationale for the specific 20% PD cap value.
- **§7.3** — the rationale for the specific 10% / 90% LGD bounds.
- **§7.4** — the rationale for the three-assertion CCF implementation.
- **§9** — the document owner's name and role.

**None of these are gaps in the notebook output.** They are the analyst's justification for design choices that the notebook implements but does not justify. The DCR records the choices; this document records the mechanism; the analyst records the reasoning.

**When the markers are filled, the document is complete.** It will be the most substantive of the four documents in this batch — the methodology document is the document that a reviewer reads to understand *what the pipeline does*, and the notebook output supports that reading fully.
