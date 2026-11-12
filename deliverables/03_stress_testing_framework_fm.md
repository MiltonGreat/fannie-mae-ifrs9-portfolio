# IFRS 9 Stress Testing Framework — Fannie Mae

**Document ID:** MAP-STF-FM-001  
**Version / Status:** 0.1 / Draft — pending analyst completion of the marked sections  
**Model under review:** Fannie Mae mortgage PD/LGD/EAD and illustrative ECL scenario pipeline  
**Framework basis:** IFRS 9 §5.5.3, §5.5.9, §5.5.11, §5.5.17, §5.5.18 (reference only; no compliance conclusion drawn)  
**Owner:** `[ANALYST: name, role]`  
**Companion documents:** [`../README.md`](../README.md) · [`../governance/methodology-document.md`](../governance/methodology-document.md) · [`../governance/decision-criteria-register.md`](../governance/decision-criteria-register.md) · [`../governance/consolidated-findings-register.md`](../governance/consolidated-findings-register.md) · [`../governance/risk-validation-plan.md`](../governance/risk-validation-plan.md)

---

## At a glance

| Field | Value |
|---|---|
| **Project** | Fannie Mae IFRS 9 ECL methodology demonstration |
| **Dataset** | Fannie Mae Single-Family Loan Performance, 100,000-row sample (seed 42) |
| **Headline metrics** | PD 0.95% · LGD 44.28% · EAD ~$204,745 · probability-weighted ECL ~$2,231 · coverage diagnostic 22.31% of assumed total assets |
| **Stage distribution** | Stage 1: ~89.1% · Stage 2: ~9.9% (SICR-by-construction) · Stage 3: ~1.0% |
| **Findings** | 13 total: 0 CRITICAL, 5 HIGH, 5 MEDIUM, 3 LOW; 13 DOCUMENTED, 0 OPEN |
| **Verdict** | Accepted as a demonstration — see [`../governance/approval-record.md`](../governance/approval-record.md) (template) |
| **What this framework is** | A methodology demonstration of the IFRS 9 ECL arithmetic |
| **What this framework is NOT** | A production-calibrated ECL model; a validated model; an IFRS 9-compliant implementation |

> **Framework purpose.** This document is the formal assessment of the Fannie Mae IFRS 9 ECL methodology: what the framework implements, what it does not implement, what the results show, and what the results do not show. **It is the FM analytical payload** — the document a reader consults to understand what the framework computes and what its limitations are.
>
> **On the relationship to the `methodology-document.md`.** The methodology document describes the *mechanism* — how PD, LGD, and EAD are constructed, how the stages are assigned, how the scenarios are weighted. This framework document is the *assessment* — what the mechanism produces, what the results mean, and what the mechanism's limitations are. **The methodology document is the specification; this framework document is the interpretation.**
>
> **On the relationship to the Lending Club project.** The Lending Club project (the companion repo) is a *model validation*. The Fannie Mae project is a *framework*. **The two ask different questions.** Lending Club asks "is this model fit for deployment?"; Fannie Mae asks "is this methodology fit to be cited as a demonstration?" **The two have different verdicts, different findings, and different artefacts.** This framework document is the Fannie Mae side of the comparison.

---

## 1. Framework scope

### 1.1 — What the framework implements

| Component | Implemented? | Evidence | Key limitation |
|---|---|---|---|
| **PD calibration** | Yes | NB03 §3 | Summary statistic, not fitted |
| **LGD calibration** | Yes | NB03 §4 | Deterministic, function of PD |
| **EAD calculation** | Yes | NB03 §5 | `ORG_UPB` base; AP-03 band breach (86.95%) |
| **Stage classification** | Yes | NB03 §3 | SICR-by-construction; C6 default×5 |
| **ECL per stage** | Yes | NB03 §6 | Single-period |
| **Probability-weighted ECL** | Yes | NB04 §4 | Scenario multipliers not calibrated |
| **Macro conditioning** | **No** | — | Multipliers are narrative, not computational |
| **Transition matrices** | **No** | — | Single-period snapshot, not projection |
| **Front-loading / cliff effects** | **No** | — | Not modelled |
| **Capital adequacy** | **No** | — | Coverage diagnostic only |

**The six "yes" rows are the arithmetic.** The framework implements PD × LGD × EAD per stage, per scenario, with discounting — the full IFRS 9 §5.5.3 arithmetic.

**The four "no" rows are the framework's scope.** It does not condition on macro variables, does not implement stage transitions over time, does not model front-loading, and does not compute capital adequacy. **These are not defects** — they are the framework's documented scope. A reader who expects a full IFRS 9 provisioning system will find the four "no" rows and be disappointed; a reader who reads the framework document's purpose statement will understand what the framework is.

### 1.2 — What the framework does not claim

**This framework does not declare IFRS 9 compliance.** The Fannie Mae pipeline computes the components IFRS 9 requires, but the pipeline's own claim-authorization discipline stops it from writing a compliance verdict. Every compliance status field in NB05 is an `[ANALYST: insert status]` placeholder. **This document is the analyst's contribution** — it presents the framework as the pipeline implements it, states the evidence, and **leaves the compliance determination to the Validation Opinion**.

**The framework does not claim production readiness.** It is a methodology demonstration. The parameters are analyst-specified, not calibrated. The results are illustrative, not a forecast. **A reader who reads the framework's headline metrics and concludes "this is what a bank would book as ECL" has misread the framework** — the framework computes what the ECL would be under the stated assumptions, not what it is.

**The framework does not claim Canadian applicability.** The pipeline is trained on U.S. data. It is not validated for Canadian portfolios. **This is a scope restriction, not a defect.** The framework's limitations document records it (L8 in the Consolidated Findings Register).

### 1.3 — What the framework's limitations are

The framework's limitations are documented in `governance/consolidated-findings-register.md` — **13 findings** (0 CRITICAL, 5 HIGH, 5 MEDIUM, 3 LOW; all 13 DOCUMENTED). The framework's `governance/` folder contains the full documentation:

- **`governance/consolidated-findings-register.md`** — the 13 findings (the `governance/` folder's findings register).
- **`governance/decision-criteria-register.md`** — the register guard criteria and their measured results.
- **`governance/methodology-document.md`** — the mechanism for each limitation.
- **`governance/risk-validation-plan.md`** — the framework's self-assessment components.
- **`governance/data-specification.md`** — the data the framework was built on and the analyst's fitness-for-purpose determination (marked `[ANALYST: ...]` in the DS).

**The framework's limitations are the framework's deliverable.** A demonstration that does not document its limitations is less useful than one that does. The framework's 13 findings are the demonstration's contribution.

---

## 2. Macroeconomic conditioning

**The pipeline does not condition on macroeconomic variables.** The scenario ECLs are driven by analyst-specified PD and LGD multipliers (1.0×/1.8×/3.0×/4.5× for PD; 1.0×/1.2×/1.5×/2.0× for LGD), not by GDP/unemployment/HPI projections. The scenario definitions include macro assumptions (GDP, unemployment, HPI) for narrative context, but those assumptions are **not inputs to the calculation**.

**This is a material limitation.** A true IFRS 9 stress test would map macro scenarios to PD/LGD via a calibrated satellite model. **The pipeline does not.** Changing the macro dict changes nothing in the ECL. The framework's §4 of the methodology document documents the mechanism; this section records the consequence.

**The framework document should state three things:**
- The scenario multipliers are **analyst-specified heuristics**, not calibrated to the portfolio.
- The macro assumptions are **narrative**, not computational.
- Recalibration to macro-conditional PD/LGD is **required** before the framework can be used for any accounting or regulatory purpose.

**Suggested remediation:** FM-R-06 in the Remediation Plan (recalibrate scenario multipliers against portfolio-specific historical data).

`[ANALYST: insert the analyst's framing of the macro limitation. The factual content above is the framework's scope. The analyst's framing is what a reader should take away from the limitation — e.g., "a framework that cannot respond to a change in the macro environment is a framework whose stress results cannot be used for capital planning." If the framing is different, say so.]`

---

## 3. Stage migration and transition matrices

**The pipeline does not implement transition matrices.** Stage classification is a single-period assignment:

- **Stage 3:** `default == 1` (the `delinquency_state >= 3` condition is equivalent, because `delinquency_state` is constructed as `default × 5` in NB01 — the C6 limitation).
- **Stage 2:** top-decile of non-Stage-3 loans by `pd_estimate` (**SICR-by-construction**, not observed credit deterioration).
- **Stage 1:** all other loans.

### 3.1 — The Stage 2 mechanism is the framework's most important interpretive point

A reader must not read "Stage 2 = 9.9%" as "9.9% of the portfolio had a significant increase in credit risk." It means "the top-decile proxy assigns 9.9% of loans to Stage 2." **The framework document should state this prominently**, and the NB03 callout (see the FM README's Stage 2 section) should carry the same language.

**Why this matters.** A bank's reported Stage 2 fraction is an *observed measurement*; this framework's is a *construction*. The distinction affects how the framework's results can be compared to a bank's actual Stage 2 numbers. **They cannot be compared directly** — the framework's Stage 2 is a proxy, not an observation.

### 3.2 — Transition matrices are required for a true IFRS 9 stress test

They model how loans migrate between stages over the stress horizon. **The pipeline does not implement this.** The framework document should state:

- The stage distribution is a **point-in-time snapshot**, not a projection.
- The stress horizon is **not modelled** — the scenario ECLs are single-period, not multi-period.
- Transition matrices and Markov-chain migration are **out of scope** for this framework and would be required for a production IFRS 9 implementation.

**Suggested remediation:** FM-R-07 (acquire current credit characteristics; implement observed SICR); a longer-horizon roadmap item for transition matrices.

`[ANALYST: insert the analyst's framing of the stage migration limitation. The factual content above is the framework's scope. The analyst's framing is what the limitation means for the framework's claims — e.g., "a single-period framework cannot answer 'what is the expected loss over the remaining life of the loan?' — it can only answer 'what is the expected loss this period?'" If the framing is different, say so.]`

---

## 4. Front-loading and cliff effects

**The pipeline does not model front-loading.** The scenario ECLs are single-period expected losses, not a projection of impairment provisions over time. The framework does not implement the IFRS 9 acceleration of impairment recognition (the "front-loading" of provisions in the first 12–24 months).

**The framework document should state:**
- The pipeline computes a **point-in-time ECL** (per-loan, per-scenario), not a **provisioning trajectory**.
- Front-loading and cliff effects are **not modelled**.
- A production IFRS 9 implementation would require a multi-period projection with stage transition.

**Why this matters for a reader.** A bank's IFRS 9 provisioning includes the front-loading effect — the recognition of lifetime losses for Stage 2 loans *before* they default. **The framework's ECL is the point-in-time loss**, not the front-loaded provision. The two numbers are different. **A reader who compares the framework's ECL to a bank's provision and finds them different has compared the wrong things.**

`[ANALYST: insert the analyst's framing of the front-loading limitation. The factual content above is the framework's scope. The analyst's framing is what a reader should understand about the comparison between the framework's ECL and a bank's provision — e.g., "the framework's ECL is the loss the bank would bear if the loans defaulted this period; the bank's provision is the loss the bank expects to bear over the lifetime of the loans. The two are not the same number." If the framing is different, say so.]`

---

## 5. Scenario design and severity

### 5.1 — What the framework's scenarios are

The pipeline implements multi-scenario probability-weighted ECL across four scenarios:

| Scenario | Weight | PD multiplier | LGD multiplier | ECL (per loan) |
|---|---|---|---|---|
| Baseline | 45% | 1.0× | 1.0× | $834.70 |
| Adverse | 30% | 1.8× | 1.2× | $1,802.96 |
| Severely Adverse | 15% | 3.0× | 1.5× | $3,756.16 |
| Tail Risk | 10% | 4.5× | 2.0× | $7,512.33 |
| **Total** | **100%** | — | — | **$2,231.16** |

**The weights sum to 1.0** (verified in NB04's config). **The multipliers are analyst-specified** based on 2008 historical analysis (L9). **The scenario design has two layers that are not connected:** macro narratives (Layer 1) documented for context; PD/LGD multipliers (Layer 2) direct inputs to the calculation. **The framework does not connect the two.**

### 5.2 — The three key caveats

**Caveat 1 — The severity check is below the target.** The Severely Adverse ECL as a rate of EAD is compared against the 2008–2010 historical average (1.44%). The pipeline reports a ratio of **1.27×** historical — **below** the 1.5× "severely adverse" threshold in the DCR. **This triggers AP-05 and IT-05.** The correct reading is: the assumed scenario multipliers produce a scenario the register considers *insufficiently severe*, not that the portfolio is safe.

**Caveat 2 — The PD cap does not bind in this run.** The Tail Risk scenario's PD is capped at 20%. In this run, the Tail Risk PD (4.29%) is below the cap, so the cap-sensitivity analysis reports **+0.0% understatement**. **The mechanism is in place** for runs where base PD is higher; **in this run it is a null result**, not a finding.

**Caveat 3 — The 3.0× multiplier is an assumption, not a calibration.** The Severely Adverse PD multiplier (3.0×) and LGD multiplier (1.5×) are **assumptions with a precedent label** — "the 2008 financial crisis" — not calibrated parameters. A validator should ask: what evidence supports 3.0× rather than 2.0× or 4.0×? The framework's answer is the precedent, not a derivation. **The framework does not validate the multipliers; it tests them against a target and reports a miss.**

**The three caveats are the framework's most consequential interpretive content.** A reader who reads only the weighted ECL ($2,231) and the scenario table will miss the three caveats. **The framework document must state them prominently** — as the FM README does in its "Notes" section.

`[ANALYST: insert the analyst's framing of the three caveats. The factual content above is the framework's scope. The analyst's framing is which caveat matters most for a reader — e.g., "the severity caveat is the framework's most important limitation: a scenario that is 1.27× the 2008 crisis is not a severely adverse scenario, and the framework's severely adverse ECL should not be cited as such." If the framing is different, say so.]`

---

## 6. Capital coverage — a note on the reframe

**The pipeline computes capital coverage, not capital adequacy.** The distinction is material:

- **Coverage:** "How much of an assumed asset base would be consumed by ECL under stress?" The ECL-as-%-of-assets metric answers this.
- **Adequacy:** "Does the bank have enough capital to survive the stress?" This requires RWA, actual CET1, and institution-specific buffers. **None of those are in the pipeline.**

The framework reports:

| Metric | Value |
|---|---|
| ECL as % of assets (weighted) | 22.31% |
| ECL as % of assets (tail) | 75.12% |
| ECL/CET1 ratio (weighted) | **495.8% — diagnostic only, not a capital adequacy finding** |

**The framework document must not present ECL/CET1 as a capital adequacy finding.** The CET1 figure is 4.5% of total assets, not 4.5% of RWA — **the ratio is a scale artifact**. The framework should present the coverage metric as the headline and the ECL/CET1 as a footnote with the scale-mismatch disclaimer.

**The reframe is the framework's most important governance lesson.** An earlier version of the framework characterized the capital result as "Inadequate under stress." **That was wrong** — the ratio was a scale mismatch, not a capital adequacy finding. **Correcting that framing is the most valuable thing the framework demonstrates about model governance.** The FM README's "The framing discipline is the point" paragraph documents this; the framework document should carry the same language.

**Suggested remediation:** FM-R-08 (EAD ratio) and FM-R-10 (PD cap disclosure) both affect the ECL-as-%-of-assets metric. Both should be resolved before the coverage metric is cited.

`[ANALYST: insert the analyst's framing of the reframe. The factual content above is the framework's scope. The analyst's framing is what a reader should take away from the coverage-vs-adequacy distinction — e.g., "a demonstration that reframes a scale artifact as a governance finding is more valuable than one that presents an inflated capital-adequacy result. The reframe is the framework's most important contribution to the analyst's practice." If the framing is different, say so.]`

---

## 7. Framework observations for the reader

The following observations are **not findings** — they are scope limitations and interpretive points the reader must understand when citing this framework.

**Observation 1 — EAD ratio outside configured band (FM-R-08).** The EAD ratio is 86.95% against a configured 50–85% band. AP-03 fired. The notebook says "check CCF first"; the notebook's own CCF=0 assertion means the cause is elsewhere. Two candidate causes: young loan sample (avg 16 months) means prepayment/curtailment factors near 1.0; `ORG_UPB` used as exposure base instead of current UPB. **The reader should not cite EAD or ECL without disclosing the ratio and the AP-03 warning.**

**Observation 2 — Stage 2 is by construction.** Stage 2 = 9.9% is the top-decile proxy, not observed SICR. **The reader must not present 9.9% as a portfolio SICR rate.** It must present it as "the top-decile proxy assigns 9.9% of loans to Stage 2."

**Observation 3 — PD cap effect is zero in the current run.** The cap (20%) does not fire at the per-loan scenario level because tail PD = 4.29% < 20%. The cap would fire if base PD rose. **The reader must disclose the cap and its conditional effect.**

**Observation 4 — Severity ratio below target.** The 1.27× ratio is below the 1.5× target. **The reader must decide whether to accept the scenario as-is** (with the IT-05 trigger documented) or require recalibration (FM-R-06).

**Observation 5 — Fairness testing incomplete.** Geographic proxies at cardinality 52/328/638 produce degenerate DI = 0.00%. **Fairness testing is incomplete, not failed.** The 4/5ths rule is misapplied at high cardinality. The correct conclusion is a data limitation.

**Observation 6 — Reconciliation warnings on `base_pd` are expected.** The dataframe's mean `pd_estimate` is 5.8%, while NB03's `unconditional_pd` is 0.95%. **These are different by design** (per-loan score vs sample default rate). The reconciliation warning is not an error.

**Observation 7 — Claim-authorization discipline.** No notebook in the pipeline writes a compliance verdict. Every status field is an analyst placeholder. **The reader should state this explicitly as a governance strength.**

**The seven observations are the framework's interpretive content.** They are not findings — they are the things a reader must know to use the framework's results correctly. **The framework document should carry them** — a framework that produces numbers without interpretive guidance is a framework that invites misreading.

`[ANALYST: insert the analyst's framing of the seven observations. The factual content above is the framework's scope. The analyst's framing is which observation matters most for a reader — e.g., "Observation 2 (Stage 2 is by construction) is the framework's most important interpretive point: a reader who cites 9.9% as observed SICR has misunderstood the framework's Stage 2 mechanism." If the framing is different, say so.]`

---

## 8. Framework summary

| Component | Implemented? | Evidence | Limitation |
|---|---|---|---|
| PD calibration | Yes | NB03 §3 | Summary statistic, not fitted |
| LGD calibration | Yes | NB03 §4 | Deterministic, function of PD |
| EAD calculation | Yes | NB03 §5 | `ORG_UPB` base; AP-03 band breach |
| Stage classification | Yes | NB03 §3 | SICR-by-construction; C6 default×5 |
| ECL per stage | Yes | NB03 §6 | Single-period |
| Probability-weighted ECL | Yes | NB04 §4 | Scenario multipliers not calibrated |
| Macro conditioning | No | — | Multipliers are narrative, not computational |
| Transition matrices | No | — | Single-period snapshot, not projection |
| Front-loading / cliff effects | No | — | Not modelled |
| Capital adequacy | No | — | Coverage diagnostic only |

### Framework sign-off

| Role | Name | Date | Signature |
|---|---|---|---|
| Model Development owner | `[ANALYST: name, role]` | `[ANALYST: date]` | `[ANALYST: signature]` |
| Validation Opinion author | `[ANALYST: name, role]` | `[ANALYST: date]` | `[ANALYST: signature]` |

> **Note on the placeholders.** The `[ANALYST: ...]` fields are the analyst's contribution, not omissions. The framework produces the evidence; the analyst produces the determination and the signature.

---

## 9. Document control

| Field | Value |
|---|---|
| **Document ID** | MAP-STF-FM-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | [`../README.md`](../README.md) · [`../governance/methodology-document.md`](../governance/methodology-document.md) · [`../governance/decision-criteria-register.md`](../governance/decision-criteria-register.md) · [`../governance/consolidated-findings-register.md`](../governance/consolidated-findings-register.md) · [`../governance/risk-validation-plan.md`](../governance/risk-validation-plan.md) · [`../governance/data-specification.md`](../governance/data-specification.md) |
| **Next review** | At the next version bump of any FM notebook |

---

## 10. What this framework contains, and what it does not

### What it contains (sourced from the FM notebook outputs and the FM `governance/` folder)

- The framework's scope: six implemented components and four not implemented.
- The macro conditioning limitation (no macro variables in the calculation).
- The stage migration limitation (single-period snapshot; SICR-by-construction).
- The front-loading limitation (point-in-time ECL, not provisioning trajectory).
- The scenario design and the three key caveats (severity below target; PD cap null result; multiplier as assumption).
- The capital coverage vs. adequacy distinction (the reframe).
- The seven framework observations.
- The framework summary table.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§2** — the analyst's framing of the macro limitation.
- **§3** — the analyst's framing of the stage-migration limitation.
- **§4** — the analyst's framing of the front-loading limitation.
- **§5** — the analyst's framing of the three caveats.
- **§6** — the analyst's framing of the reframe.
- **§7** — the analyst's framing of the seven observations.
- **§8** — the framework sign-off (the two signature blocks).
- **§9** — the document owner's name and role.

**The document's structure and factual content are sourced.** The analyst's framing of each limitation — *why it matters, what a reader should take away from it* — is the analyst's contribution, marked in the six sections where the framing belongs.

**When the markers are filled, the document is complete.** It is the FM analytical payload — the document the FM framework is assessed on, and the document a reader consults to understand what the framework computes and what it does not.
