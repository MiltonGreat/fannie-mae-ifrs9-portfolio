# Fannie Mae IFRS 9 ECL Demonstration — Feature Decision Log

**Document ID:** FM-FDL-001  
**Version / Status:** 1.1 / Frozen at NB01 v1.2 execution (run_id `20261021_121434`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Feature Decision Log (FDL)  
**Source feature inventory:** `data/feature_inventory.csv` (created in NB01 §6)  
**Companion documents:** FM-PDR-001 (Problem Definition Record) · FM-DS-001 (Data Specification) · FM-MD-001 (Methodology Document) · FM-RVP-001 (Risk & Validation Plan) · FM-DCR-001 (Decision Criteria Register) · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan)

---

## At a glance

| Field | Value |
|---|---|
| **Raw columns (post-sampling)** | 133 before construction; 175 after construction and missing-indicators |
| **Target** | `default` (binary) |
| **Target transforms (excluded)** | 4 — `delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag` |
| **Constructed risk drivers** | 7 — `pd_estimate`, `lgd_final`, `ead_final`, `upb_ratio`, `delinquency_state`, `risk_fico`, `risk_ltv` |
| **Constructed parameters used in ECL** | 3 — `pd_estimate`, `lgd_final`, `ead_final` |
| **Missing-indicator columns** | 10 (created in NB01) |
| **Bookkeeping columns** | 2 (`run_id`, `notebook_version`) |
| **Known limitation** | `delinquency_state = default × 5` (C6) — Stage 3 driven by target, not observed delinquency |

> **Document purpose.** This log records, for every feature in the Fannie Mae dataset, the decision to include, exclude, transform, or construct it — and the reason. Unlike the Lending Club FDL, the Fannie Mae pipeline has **two distinct feature decisions**:
>
> 1. **Features used for the observed target (`default`)** — standard include/exclude rules.
> 2. **Features used for the constructed parameters (`pd_estimate`, `lgd_final`, `ead_final`)** — these are **specified, not fitted**, so the log must document the construction as a feature decision.
>
> The second category is the FDL's most important role in the Fannie Mae pipeline: **it is the document that says `pd_estimate` is not a model output — it is an analyst-specified formula, and here are the coefficients.**

---

## 1. Purpose and scope of the log

This log records, for every feature in the Fannie Mae dataset, the decision to include, exclude, transform, or construct it — and the reason. Unlike Lending Club, the Fannie Mae pipeline has two distinct feature decisions:

- **Features used for the observed target (`default`)** — the standard include/exclude rules.
- **Features used for the constructed parameters (`pd_estimate`, `lgd_final`, `ead_final`)** — these are specified, not fitted, so the log must document the construction as a feature decision.

This is the FDL's most important role in the Fannie Mae pipeline: **it is the document that says `pd_estimate` is not a model output — it is an analyst-specified formula, and here are the coefficients.**

**The distinction from LC-FDL-001.** The Lending Club FDL's central concern was leakage — nine post-origination features that had to be excluded. The Fannie Mae FDL's central concern is **constructed parameters** — seven analyst-specified formulas (`pd_estimate`, `lgd_final`, `ead_final`, `upb_ratio`, `delinquency_state`, `risk_fico`, `risk_ltv`) that the pipeline treats as inputs. The two FDLs have different primary jobs because the two projects have different primary risks.

---

## 2. Feature inventory overview

| Metric | Value |
|---|---|
| Raw columns in the 100K sample | 133 |
| Columns after construction and missing-indicators | 175 |
| Bookkeeping (dropped from features) | 2 (`run_id`, `notebook_version`) |
| Target | 1 (`default`) |
| Target transforms (excluded from features) | 4 (`delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag`) |
| Missing-indicator columns | 10 |
| Constructed risk drivers | 7 (`pd_estimate`, `lgd_final`, `ead_final`, `upb_ratio`, `delinquency_state`, `risk_fico`, `risk_ltv`) |
| Final cleaned shape | 100,000 × 175 |

**On the count.** The 175-column figure is **post-construction**: the raw CSV has 133 columns, and NB01 adds the seven constructed columns and the ten missing-indicator columns, plus the two bookkeeping columns. **The FDL's decision set includes the constructed columns**, because a constructed column is a feature decision (whether to construct it, and how), just as an include/exclude decision is.

**On the "include decisions" count.** The v1.0 FDL left this as `[ANALYST: depends on the feature set used]`. The v1.1 FDL does not attempt a single count, because the number depends on what counts as an "include" — a raw column, a constructed column, a missing-indicator column. §7 enumerates the decisions by category (raw features used, constructed features, excluded features) so a reader can see the actual decision set.

---

## 3. Inclusion criteria and rules

A feature is included if:

1. **It is available at origination** (not post-origination information — though Fannie Mae has less of this than Lending Club).
2. **It is not a target transform** (`delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag` are excluded).
3. **It is not a bookkeeping column** (`run_id`, `notebook_version`).
4. **It is not an identifier with no predictive value.**

**The "target transform" rule is Fannie Mae-specific.** Lending Club's leakage was *post-origination information* (paid amounts, balance, months since delinquency). Fannie Mae's leakage is *derived from the target*. `delinquency_state = default × 5` is the most important case — it would correlate 1.0 with `default` and would drown out every real feature in the correlation matrix.

---

## 4. Exclusion criteria and rules

Applied in order. Once a rule fires, the feature is excluded and later rules do not matter.

| Order | Rule | What it catches | Example |
|---|---|---|---|
| 1 | **Target transform** | Derived from `default`; would produce trivial correlation | `delinquency_state`, `delinquency_label`, `ifrs9_stage`, `default_flag` |
| 2 | **Bookkeeping** | Provenance columns, not predictive | `run_id`, `notebook_version` |
| 3 | **No predictive value** | Constant, identifier-like, or clearly irrelevant | `YEARMONTH` (vintage, not risk driver) |
| 4 | **Redundant** | Collinear with another feature by construction | `risk_fico` (monotone transform of `FICO_BOR`) |

**On the `risk_fico` exclusion.** This is D022 in the decision log: `risk_fico` is a monotone transform of `FICO_BOR`, so including both is redundant. The same applies to `risk_ltv` (a monotone transform of `ORG_LTV`). The exclusion is deliberate — the raw feature (`FICO_BOR`) is the interpretable input to `pd_estimate`, and the sigmoid-transformed version (`risk_fico`) adds no information that the raw feature does not already contain, except through the transform's *shape*, which the pipeline captures via the `pd_estimate` formula's own scaling.

**Why `risk_fico` and `risk_ltv` are still constructed even though they are excluded.** They are built in NB01 as intermediate values (the sigmoid transform is the first step of the `pd_estimate` formula). They are excluded from the *feature matrix* that a downstream model would use, but they exist in the cleaned dataframe because `pd_estimate` depends on them. A reader who sees `risk_fico` in the cleaned dataframe's columns but not in the feature matrix should understand that it is a constructed intermediate, not an excluded feature that was accidentally left in.

---

## 5. Transformations and construction

### 5.1 — Constructed risk drivers (NB01 Phase 2)

| Feature | Construction | Why constructed |
|---|---|---|
| `pd_estimate` | Weighted combination of FICO, LTV, DTI, AGE; normalized; scaled to [0.01, 0.20] | The model's PD is **not fitted** — it is specified. The formula is in §5.2, not the code. |
| `lgd_final` | Base 45% + age/LTV/property adjustments; clipped to [0.10, 0.90] | LGD is **specified, not fitted**. Adjustments are in §5.3. |
| `ead_final` | `ORG_UPB` × prepayment_factor × curtailment_factor, CCF = 0 | EAD is a **deterministic formula**. CCF is asserted to 0 (C4 discipline). |
| `upb_ratio` | `CUR_UBP / ORG_UPB` | Relative remaining balance — used for EAD calibration. |
| `delinquency_state` | `default × 5` | IFRS 9 stage proxy — but it is a **target transform** (C6 limitation). |
| `risk_fico` | Sigmoid transform of `FICO_BOR`, inflection at 720 | Ordinal risk score; **excluded from prediction** (D022 — redundant). |
| `risk_ltv` | Sigmoid transform of `ORG_LTV`, inflection at 80 | Ordinal risk score; **excluded from prediction** (D022 — redundant). |

### 5.2 — `pd_estimate` formula

The `pd_estimate` formula is the FDL's most important content. **Every coefficient and every bound is a feature decision.** Changing any of them is a **MAJOR** version bump and requires re-running NB03, NB04, and NB05.

risk_score = (1 - (FICO_BOR - 620) / 212) × 0.4

    (ORG_LTV / 100) × 0.3

    (DTI / 50) × 0.2

    (AGE / 36) × 0.1

risk_score = risk_score / max(risk_score)

pd_estimate = clip(0.01 + 0.09 × risk_score, 0.01, 0.20)


**On the coefficients.** The weights (0.4, 0.3, 0.2, 0.1) are analyst-specified based on credit-risk theory: FICO is the strongest single predictor, LTV is the second, DTI and AGE are weaker. The scaling bounds (0.01, 0.20) are the PD floor and cap. **These are not fitted coefficients** — a production model would estimate them via logistic regression on the observed `default` target. Here they are specified because the goal is methodology demonstration, not calibration quality.

**On the normalization.** `risk_score / max(risk_score)` divides by the maximum value in the sample, so the score is scaled to [0, 1] before the `0.01 + 0.09 × risk_score` scaling. This is why `pd_estimate`'s range is [0.01, 0.10] in practice — the formula `0.01 + 0.09 × 1.0 = 0.10` is the maximum before the `clip(0.01, 0.20)` bound. **The `clip(0.01, 0.20)` bound only fires if the sample's normalization is bypassed** — in the current run, `pd_estimate` values are in [0.01, 0.10], and the cap at 0.20 is not reached. A reader who sees "range [0.01, 0.20]" and then looks at the actual `pd_estimate` values should understand that the reported range is the specification's bound, not the current run's observed range.

### 5.3 — `lgd_final` construction

`lgd_final` is built as `base_lgd + age_adjustment + ltv_adjustment + property_adjustment`, clipped to [0.10, 0.90].

| Adjustment | Values | Source |
|---|---|---|
| Base LGD | 45% | Config (`base_lgd = 0.45`) |
| Age adjustment | +10% for 0–12m, +5% for 12–24m, 0% for 24–36m, −5% for 36–60m, −10% for 60–120m, −15% for 120m+ | LGDCalibrator's `age_adjustments` dict (NB03 Phase 4) |
| LTV adjustment | −10% for Low, 0% for Medium, +15% for High | LGDCalibrator's `ltv_adjustments` dict |
| Property adjustment | SF: −5%, PU: 0%, CO: +10%, MH: +20%, CP: +15% | LGDCalibrator's `property_adjustments` dict |
| Floor / cap | 10% / 90% | Config (`lgd_floor`, `lgd_cap`) |

**On the adjustments.** Every adjustment is a feature decision. The values are analyst-specified based on collateral-segmentation reasoning (younger loans have less equity → higher LGD; high-LTV loans have less collateral cushion → higher LGD; mobile homes lose value fastest → highest LGD). A production model would fit these adjustments against observed recovery data; here they are specified because the goal is methodology demonstration.

**On the `lgd_final` range.** The base 45% plus the adjustments places most loans in the [30%, 60%] range, which is the DCR's target band for LGD. The floor (10%) and cap (90%) are safety bounds that fire on extreme combinations (e.g., a very young mobile-home loan would hit the cap; a very old single-family loan would hit the floor).

---

## 6. Leakage and target-transform exclusions

### 6.1 — No post-origination leakage in the Lending Club sense

**The Fannie Mae dataset does not contain the equivalent of Lending Club's `paid_total` / `balance`.** The "current state" (`CUR_UBP`, `AGE`, `YEARMONTH`) is what the pipeline's *target* is derived from — the target is a snapshot of the loan's status, and the pipeline does not use status fields as features (see FM-DS-001 §6). The dataset is origination-plus-current-state, and current-state fields are excluded from the feature matrix.

**The consequence for the leakage audit.** The Fannie Mae pipeline's leakage risk is **not** the Lending Club leakage risk (post-origination outcomes used as features). The leakage risk is **target transforms** — fields derived from `default` that would produce trivial correlations if used as features. The next subsection enumerates those.

### 6.2 — Four target transforms (excluded from correlation and prediction, per FDL §1.3)

| Feature | Why excluded |
|---|---|
| `delinquency_state` | Constructed from `default` (`default × 5`) — would correlate 1.0 with the target |
| `delinquency_label` | Derived from `delinquency_state` — same issue |
| `ifrs9_stage` | Derived from `default` in NB03 — target transform |
| `default_flag` | Derived from `default` — target transform |

**The `delinquency_state` case is documented as C6.** It is the most important target transform in the Fannie Mae pipeline, because it means **Stage 3 in the IFRS 9 calculation is effectively "the loan defaulted", not "the loan is delinquent."** A reader of the FDL should understand this before reading the IFRS 9 staging logic in the Methodology Document. See MAP-STF-FM-001 §3 for the mechanism.

**Why these are excluded from correlation.** The FDL's §1.3 rule excludes target transforms from the correlation matrix because a correlation of 1.0 with the target would drown out every real feature's correlation. The exclusion is a methodological necessity, not an aesthetic choice — including `delinquency_state` in the correlation matrix would make every other feature's correlation look negligible by comparison.

---

## 7. Feature-by-feature decisions

### 7.1 — Included (raw features used for PD estimation or ECL)

| Feature | Type | Decision | Reason |
|---|---|---|---|
| `FICO_BOR` | Numeric | Include | Strong borrower credit signal; used in `pd_estimate` |
| `ORG_LTV` | Numeric | Include | Collateral position; used in `pd_estimate` and `lgd_final` |
| `DTI` | Numeric | Include | Debt burden; used in `pd_estimate` |
| `AGE` | Numeric | Include | Seasoning; used in `pd_estimate` and `lgd_final` |
| `ORG_UPB` | Numeric | Include | Loan size; used in `ead_final` |
| `ORG_CLTV` | Numeric | Include | Combined LTV; used in LGD adjustments |
| `PROP_TYPE` | Categorical | Include (via mapping) | Property type; used in LGD adjustments |
| `OCCU_STAT` | Categorical | Include (via mapping) | Occupancy; possible signal |

**On the included features.** The eight features here are the ones explicitly used in the constructed parameters (`pd_estimate`, `lgd_final`, `ead_final`). A production model would likely include more origination characteristics (e.g., `NUM_BOR`, `ORIG_RATE`, loan purpose) as direct predictors of `default`. The current pipeline's feature set is deliberately small because the goal is to demonstrate the ECL methodology, not to build the strongest possible `default` predictor.

### 7.2 — Constructed (frozen in the FDL)

| Feature | Role | Notes |
|---|---|---|
| `pd_estimate` | Input to ECL | Formula in §5.2; coefficients frozen |
| `lgd_final` | Input to ECL | Base + adjustments in §5.3; frozen |
| `ead_final` | Input to ECL | Deterministic formula; CCF = 0 enforced |
| `upb_ratio` | Input to EAD calibration | `CUR_UBP / ORG_UPB` — a relative balance, used only as an intermediate |
| `delinquency_state` | IFRS 9 stage proxy | **Target transform** (C6) — excluded from features; used only as stage assignment |
| `risk_fico` | Intermediate for `pd_estimate` | **Excluded from prediction** (D022) — redundant with `FICO_BOR` |
| `risk_ltv` | Intermediate for `pd_estimate` | **Excluded from prediction** (D022) — redundant with `ORG_LTV` |

### 7.3 — Excluded

| Feature | Reason | Decision ID |
|---|---|---|
| `delinquency_state` | Target transform | FDL §1.3 (C6) |
| `delinquency_label` | Derived from `delinquency_state` | FDL §1.3 |
| `ifrs9_stage` | Derived from `default` in NB03 | FDL §1.3 |
| `default_flag` | Derived from `default` | FDL §1.3 |
| `risk_fico` | Redundant (monotone transform of `FICO_BOR`) | D022 |
| `risk_ltv` | Redundant (monotone transform of `ORG_LTV`) | D022 |
| `YEARMONTH` | Vintage; not a risk driver at origination | Implicit (no explicit decision ID; see §8.1) |
| `run_id`, `notebook_version` | Bookkeeping | Implicit (no explicit decision ID; see §8.1) |

**On the "implicit" decisions.** `YEARMONTH` and the bookkeeping columns are excluded without an explicit decision ID. The v1.0 FDL left them implicit; the v1.1 FDL flags them as implicit but does not assign IDs. **A future version could assign them decision IDs** (e.g., D026 for `YEARMONTH`, D027 for bookkeeping) so every exclusion has an explicit ID. This is a documentation improvement, not a substantive change.

---

## 8. Open items and dependencies

### 8.1 — Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **`pd_estimate` formula** | Analyst-specified. A production version would fit coefficients via logistic regression on the observed `default` target. | Not in the 6-month remediation plan. The FDL's largest open item. |
| 2 | **`delinquency_state = default × 5` (C6)** | The C6 limitation. A true IFRS 9 model would use observed delinquency buckets. Currently the pipeline collapses Stage 3 to "defaulted", which is a conservative simplification. | MAP-RP-001 · FM-R-07 (data acquisition for observed delinquency). |
| 3 | **`risk_fico` and `risk_ltv`** | Constructed but excluded from prediction. If a future model wanted to use them (e.g., for monotone constraints in XGBoost), the decision would change. | Not in the current roadmap. Documented here as a candidate future-version change. |
| 4 | **Macro features** | No macro indicators in the dataset. If macro data were joined, the FDL would need to document the join and the lag structure. | MAP-RP-001 · FM-R-06 (scenario calibration; macro overlay). |
| 5 | **Implicit decisions** | `YEARMONTH` and bookkeeping columns are excluded without explicit decision IDs. | Documentation improvement candidate. |

**On the "not in the 6-month remediation plan" status.** Items 1 and 3 are documented as FDL open items but are not part of the 6-month remediation plan. Item 1 (the `pd_estimate` formula) is not a defect — it is a documented scope limitation (the pipeline is a methodology demonstration, and the formula is specified, not fitted). Item 3 (`risk_fico` / `risk_ltv`) is a candidate future-version change, not a current defect.

### 8.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | FM-DS-001 (data specification), FM-MD-001 (methodology), FM-AS-001 (analytical spec), FM-DCR-001 (decision criteria) |
| Downstream | NB02 (uses features), NB03 (calibrates on them), NB04 (stress tests), NB05 (compliance), NB06 (Power BI export) |
| Reproducibility | `feature_inventory.csv` and `notebook_03_parameters.json` are the frozen contracts |

---

*End of document.*
