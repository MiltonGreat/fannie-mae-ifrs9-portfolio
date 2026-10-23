
**The critical pipeline property.** NB03 writes `notebook_03_parameters.json` as a frozen contract. NB04, NB05, and NB06 read it — **they do not recompute**. Changing a parameter in NB03 invalidates all downstream artifacts, which is why the parameter JSON is the **reproducibility anchor**.

**On the "frozen contract" discipline.** The contract is enforced at runtime: a downstream notebook that reads the JSON asserts the presence of the fields it needs, and a notebook that tries to re-derive a parameter from the dataframe is caught by the reconciliation check (NB04 Phase 2 and NB05 Phase 2 both log a warning if the dataframe's value diverges from the contract). The contract is the specification's enforcement mechanism, not just a file.

---

## 4. Parameter specification (frozen values)

All parameters are in `IFRS9Config` (NB03 Phase 1, dataclass). **This section is the methodology's parameter reference** — every value is documented with its rationale. The current run's measured values against the register guards are in FM-DCR-001 §4; the parameter *values* themselves are the methodology's specification, not a run result.

### 4.1 — Parameter table

| Parameter | Value | Rationale |
|---|---|---|
| `pd_observation_window` | 12 months | IFRS 9 12-month PD requirement (Stage 1) |
| `mortgage_term_years` | 30 | Standard mortgage term |
| `stage1_threshold` | 0.005 | 0.5% — Stage 1 boundary |
| `stage2_threshold` | 0.05 | 5% — Stage 2 boundary |
| `use_pit_pd` | True | Point-in-time PD (not through-the-cycle) |
| `pd_floor` | 0.0001 | 0.01% floor |
| `pd_cap` | 0.20 | 20% cap — masks tail convexity (L6) |
| `base_lgd` | 0.45 | 45% base recovery loss |
| `lgd_floor` | 0.10 | 10% LGD floor |
| `lgd_cap` | 0.90 | 90% LGD cap |
| `prepayment_rate` | 0.08 | 8% annual prepayment |
| `curtailment_rate` | 0.02 | 2% annual curtailment |
| `ccf` | 0.0 | **Term mortgages have no undrawn commitment** (C4 discipline) |
| `eir` | 0.035 | 3.5% effective interest rate |
| `sicr_pd_increase_threshold` | 2.0 | 2× PD increase triggers SICR |
| `sicr_fico_drop_threshold` | 50 | 50-point FICO drop triggers SICR |
| `sicr_delinquency_backstop` | 30 | 30 DPD backstop |

### 4.2 — The two most consequential parameters

**`ccf = 0.0` — enforced at three boundaries.** The CCF (credit conversion factor) is asserted to be exactly 0.0 at three places:

1. **Config creation** (`IFRS9Config` initialization) — an `assert config.ccf == 0.0` after instantiation.
2. **Computation** (EADCalculator's `calculate_ead`) — an `assert self.config.ccf == 0.0` before the EAD arithmetic.
3. **Return** (`ead_estimates['ccf_used'] == 0.0`) — an assertion on the returned value.

**The three assertions are belt-and-suspenders.** A nonzero CCF for a term mortgage is a specification violation — a CCF applies only to revolving facilities with undrawn commitments, and a term mortgage has no undrawn commitment. **Any CCF > 0 anywhere fails loudly.** The three assertions ensure that a code change that re-introduces a nonzero CCF is caught at the first opportunity, not at a downstream ECL calculation.

**`pd_cap = 0.20` — the cap fires on this run.** The lifetime PD is `1 - (1 - 0.0095)^30 ≈ 24.9%`, clipped to 20%. **The cap fires on this run.** This is the mechanism behind **L6** — the PD cap masks tail convexity, and NB04 quantifies the pre-cap understatement (see FM-DCR-001 §4.6 for the measured cap effect).

**Why the cap exists.** Without the cap, the lifetime PD for a portfolio with a 1% unconditional PD would be 24.9%, and the tail-risk scenario (4.5× PD multiplier) would produce a PD above 100% before the per-scenario clip. The cap is a stability measure — it prevents the lifetime PD from exceeding a bound that is defensible as "an upper bound on lifetime default probability for a performing mortgage." The cap is **not** a calibration result; it is a methodology choice, and the analyst must justify it in the framework's observations (see MAP-STF-FM-001 §7).

---

## 5. Calibration methodology

**There is no training.** The parameters are frozen in the config; the pipeline calibrates PD/LGD/EAD deterministically from the data. This section describes the calibration steps and the evaluation criteria.

### 5.1 — Calibration steps

| Step | What happens | Where |
|---|---|---|
| **PD** | Unconditional PD = mean of `default`, clipped to [`pd_floor`, `pd_cap`]. Lifetime PD = `1 - (1 - pd)^30`, clipped to [`pd_floor`, `pd_cap`]. | NB03 Phase 3 |
| **Stage classification** | Stage 3 = `delinquency_state >= 3` (i.e., `default`); Stage 2 = top-decile of non-Stage-3 by `pd_estimate`; Stage 1 = remaining. | NB03 Phase 3 |
| **LGD** | Base 45% + age / LTV / property adjustments, clipped to [`lgd_floor`, `lgd_cap`]. | NB03 Phase 4 |
| **EAD** | `ORG_UPB` × prepayment_factor × curtailment_factor, with CCF = 0 enforced at three boundaries. | NB03 Phase 5 |
| **ECL** | PD × LGD × EAD × discount, per stage; probability-weighted ECL across four scenarios. | NB03 Phase 6 |

### 5.2 — The staging mechanism

**Stage 3 is driven by the target, not by observed delinquency.** The `delinquency_state` column is constructed as `default × 5` in NB01, so the Stage 3 condition `delinquency_state >= 3` is always equivalent to `default == 1`. **This is the C6 limitation** — Stage 3 is effectively "the loan defaulted", not "the loan is delinquent." A reader who interprets the Stage 3 fraction as an observed delinquency rate has misread it.

**Stage 2 is SICR-by-construction.** The Stage 2 population is the top decile of non-Stage-3 loans by `pd_estimate`. This produces a Stage 2 fraction of approximately 10% by construction, not by observed credit deterioration. **This is L2** and **F-07 in the findings register**. **A reader must not read "Stage 2 = 9.9%" as "9.9% of the portfolio had a significant increase in credit risk."** It means "the top-decile proxy assigned 9.9% of loans to Stage 2."

**Why the staging is set up this way.** A true IFRS 9 SICR assessment requires current credit characteristics (current FICO, current delinquency, current utilisation) to compare against origination. The Fannie Mae dataset contains origination characteristics and a snapshot of current status, but not a time-series of current characteristics. **The proxy is the best available approach on this data**; it is documented as a limitation, not as a defect of the methodology.

### 5.3 — Evaluation: the register guards

The ECL is evaluated against the AP-02/AP-03/AP-04 register guards. **The register guards are the evaluation** — they flag when a parameter is outside its expected band, and the analyst investigates. They do not change the parameters.

| Guard | Band | What it checks |
|---|---|---|
| AP-02 | LGD 40–60% | Average LGD is inside the target band |
| AP-03 | EAD ratio 50–85% | Average EAD / ORG_UPB is inside the target band |
| AP-04 | ECL ratio of EAD 1–3% | Probability-weighted ECL / average EAD is inside the target band |

**The "detection, not correction" discipline.** The register guards fire based on thresholds, not on analyst judgment. A guard that fires is a signal for investigation; a guard that does not fire is **not** evidence that the parameter is correct — it is evidence that the parameter is inside the DCR's expected band. The distinction matters: a parameter can be inside the band and still be wrong (e.g., because the band itself is too wide); the analyst must investigate the parameter's *construction*, not just its *band membership*.

**Where the measured values are.** The current run's values against each guard are in **FM-DCR-001 §4** (measured thresholds) and in **MAP-STF-FM-001 §7** (framework observations). The v1.3 MD stated the current values ("44.28% is inside", "86.95% is outside", "1.09% is inside") as if they were part of the methodology; the v1.4 MD cites the DCR for these values.

---

## 6. Fairness and calibration methodology

### 6.1 — Fairness: not tested, documented as L7

**Fairness is not tested.** The Fannie Mae dataset has no protected attributes, and no region-based proxy is constructed in the current pipeline. This is **L7** in the limitations register. A production IFRS 9 implementation would need to address fairness, but the demonstration scope is parameterization and staging, not fairness.

**Why no region-based proxy.** Unlike Lending Club (where state was used as a region proxy), the Fannie Mae pipeline does not construct a region-based fairness test. The reasons: (a) the dataset's geography fields are origination-time (loan's property state is fixed at origination), and a region-based DI on origination geography would not be a meaningful fairness test — it would measure regional origination patterns, not model behaviour; (b) the Fannie Mae dataset's current-status fields are what the model predicts, not what it decides, so "fairness of the model's decisions" is not the right framing for an ECL calculator. The framework's observations (MAP-STF-FM-001 §7) state this.

### 6.2 — Calibration: the LGD adjustments

**The LGD adjustments (age/LTV/property) are the calibration.** They are analyst-specified, not fitted, and documented in the FDL (FM-FDL-001 §5.3). The adjustments are the model's only "calibration" in the statistical sense — they are the values that determine how LGD varies across loans.

**On the term "calibration".** The MD uses "calibration" in two senses: (a) the PD calibration (mean default, clip, stage assignment) and (b) the LGD "calibration" (the adjustment values). The two are not the same kind of operation — the PD calibration is a summary statistic computation; the LGD calibration is an analyst-specified function. **The MD should distinguish them**, because a reader who treats both as "calibration" may think the LGD adjustments are fitted. They are not.

---

## 7. Validation methodology

### 7.1 — The Fannie Mae validation paradigm

**The Fannie Mae pipeline does not have an NB03-equivalent "independent validation" notebook.** The validation happens in the Model Analysis Package — a separate document authored by the validator, not a notebook.

**This is a deliberate design difference from Lending Club.** The two projects have different validation paradigms:

| Aspect | Lending Club | Fannie Mae |
|---|---|---|
| **Verdict authoring** | NB03 (a notebook) | The framework document (MAP-STF-FM-001) |
| **Verdict contract** | `performance_summary_v2.json` | `notebook_03_parameters.json` + the framework document |
| **Verdict ownership chain** | NB02 → NB03 → NB04 → NB05 | NB03 → framework document → Methodology Owner |
| **Independent recomputation** | NB03 recomputes from predictions | The framework document recomputes from parameters |

**Why the difference.** Lending Club is a **model validation** — "is the model fit for deployment?" The verdict lives in the pipeline because the pipeline is the model's home. Fannie Mae is a **methodology validation** — "is the methodology defensible?" The verdict lives in the framework document because the methodology is the pipeline's home, and the framework document is where the methodology is assessed.

**On the claim-authorization discipline.** The Fannie Mae pipeline's claim-authorization discipline (no notebook writes a compliance verdict; every status field is an `[ANALYST: ...]` placeholder) is the mechanism that keeps the verdict out of the notebooks. The framework document is where the verdict is written — by the analyst, not by the pipeline. **A reader who reads the Fannie Mae notebooks and looks for a verdict will not find one, and should not expect to find one.** The verdict is in the framework document.

### 7.2 — What the framework document does

The framework document (MAP-STF-FM-001) reads `notebook_03_parameters.json`, recomputes per-stage and probability-weighted ECL, checks the AP-02/AP-03/AP-04 register guards, assesses the SICR proxy, the PD cap, and the EAD young-portfolio effect, and produces a verdict on **the methodology, not a model**.

**On the framework document's relationship to this MD.** The MD states *what the methodology is*; the framework document states *what the methodology's observations are*. The MD's limitations (L1–L10) are the framework document's starting point; the framework document's observations are the MD's limitations, assessed. **The two documents are paired, not interchangeable** — a reader who wants the methodology reads the MD; a reader who wants the verdict reads the framework document.

---

## 8. Known methodological limitations and open items

### 8.1 — Limitations (L1–L10)

The methodology's limitations are the same L1–L10 that appear in FM-DS-001 §8 and in the framework observations. **This MD does not restate them in full**; the authoritative table is in FM-DS-001 §8. The MD's role is to state which limitations are *methodological* (as opposed to data limitations).

**Methodological limitations (the MD's specific concern):**

| ID | Limitation | Methodological consequence |
|---|---|---|
| L1 | PD-LGD correlation is constructed | The convexity between PD and LGD is illustrative, not empirical — the ECL calculation does not reflect a recovery-data-calibrated correlation |
| L2 | SICR uses top-decile proxy | Stage 2 ECL is a construction, not an observation — the Stage 2 fraction reflects the proxy, not observed credit deterioration |
| L6 | PD cap (20%) masks tail convexity | The lifetime PD is capped, and the Tail Risk scenario's ECL is understated pre-cap — NB04 quantifies the understatement |
| L9 | Scenario weights and multipliers are assumptions | The probability-weighted ECL is a weighted sum of analyst-specified scenarios, not calibrated to history |

**On the exclusion of L3, L4, L5, L7, L8, L10 from this list.** Those limitations are also present, but they are **data or scope limitations** (documented in FM-DS-001 §8), not methodological limitations. L3 (LGD is a function of PD) is a methodological choice, but it is documented in the FDL, not the MD; L4 (prepayment/curtailment assumptions) is a parameter choice, documented in §4; L5 (EIR is industry average) is a parameter choice, documented in §4; L7 (fairness not tested) is a scope limitation, documented in §6; L8 (U.S. data) is a data limitation; L10 (capital coverage is a diagnostic) is a framing choice, documented in the framework document.

**On the relationship between the MD's limitations and the DCR's.** The DCR's §4 (in the Lending Club convention) reports measured values against criteria. The Fannie Mae DCR's equivalent section reports measured values against the register guards (AP-02/03/04) and against the framework's observations. **The MD's limitations are the *prescribed* methodology's limitations; the DCR's §4 is the *measured* results.** A reader who sees a limitation in the MD and does not see a corresponding finding in the DCR should not assume the limitation is unmeasured — the DCR measures what the register guards can measure, and the framework document assesses the rest.

### 8.2 — Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **The PD cap** | 20% is a conservative cap, but it masks tail convexity. A production version should either remove the cap (with a separate tail-risk analysis) or justify 20% explicitly. | MAP-RP-001 · FM-R-10 (monitoring); longer-horizon candidate for cap recalibration |
| 2 | **The SICR proxy** | Top-decile is a simplification. A production version would use observed credit characteristics (delinquency, recent FICO changes) to identify true SICR. | MAP-RP-001 · FM-R-07 (data acquisition) |
| 3 | **The young-portfolio effect** | AP-03 fires because the portfolio is young (avg age 16 months, mostly 2021–2023 vintages). This is a data limitation, not a methodology defect, but it means the EAD ratio is not representative of a seasoned book. | MAP-RP-001 · FM-R-08 (EAD ratio diagnosis); documented in FM-DS-001 §6.2 |
| 4 | **Fairness testing** | Not tested. A production IFRS 9 implementation would need to test fairness on protected attributes (which the Fannie Mae dataset does not have — a fundamental data limitation). | MAP-RP-001 · FM-R-09 (data acquisition); longer-horizon item |

**On the routing.** Items 1, 2, 3, and 4 are routed to the Remediation Plan (MAP-RP-001) with specific finding IDs, or documented as longer-horizon items where the remediation is out of the 6-month plan's scope. The MD's open items are the methodology's open items; the Remediation Plan's findings are the closure work.

---

*End of document.*
