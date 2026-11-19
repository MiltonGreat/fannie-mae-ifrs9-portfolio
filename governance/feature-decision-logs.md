# Feature Decision Log (A04)

| Metadata Field | Value |
| :--- | :--- |
| **Document ID** | `[ANALYST: Document ID — cross-referenced elsewhere as FM-FDL-001]` |
| **Version** | `0.1.0-draft` (Forward-Looking Planning Draft) |
| **Run ID / Timestamp** | `YYYYMMDD_HHMMSS` *(literal placeholder — link to active execution run log on run)* |
| **Author / Owner** | `[ANALYST: name, role]` |
| **Project Scope** | IFRS 9 Expected Credit Loss (ECL) Methodology Demonstration (Public Data) |
| **Artifact Type** | Planning document — records feature decisions to be built, not a run record |
| **Regulatory Alignment References** | IFRS 9, OSFI E-23, SR 26-2, ECOA *(illustrative/defensibility alignment; not a compliance certification)* |
| **Cross-Referenced Artifacts** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03), A05 (Methodology Document), `FM-DCR-001` (A07), `FM-CFR-001`, `feature_inventory.csv` |

> **Governance & Scope Notice:** This document is a forward-looking planning record of feature decisions for the IFRS 9 ECL methodology demonstration on public Fannie Mae data. It records what the analyst intends to build and why, and explicitly marks decisions that are deferred to a named artifact/notebook and owner. It is not a run record; measured values and results belong to the execution artifacts (A11 / A07). It is not a production model specification and not an independent validation report.

> **Status-line convention:** Each of the six sections ends with a status line in one of three states:
> - `Decided` — the decision is made and recorded here.
> - `Deferred to [artifact/notebook] — owner: [role]` — the decision is intentionally not made here; it will be made in the named artifact by the named owner.
> - `Open — decision needed by [milestone]` — no decision exists yet and one is required by a stated milestone.

> **Schema statement (A04 schema):** This log is organized around six questions. Each is a required section: (1) inclusion/exclusion, (2) transforms, (3) proxies and their gaps, (4) business meaning, (5) stability, (6) fairness implications.

---

## 1. Which features are included, and which are excluded?

### 1.1 — Named exclusions (decided)

| Feature | Decision | Rationale |
| :--- | :--- | :--- |
| `risk_fico` | EXCLUDE | Monotone transform of FICO; carries no independent information. `[VERIFY: confirm `risk_fico` is a column in `train4.csv` in NB01 Phase 1; if a constructed/renamed name, record the source column]` |
| `risk_ltv` | EXCLUDE | Monotone transform of LTV; carries no independent information. `[VERIFY: confirm `risk_ltv` is a column in `train4.csv` in NB01 Phase 1; if a constructed/renamed name, record the source column]` |
| `pd_estimate` | EXCLUDE from feature matrix | Constructed from FICO, LTV, DTI, AGE; collinear with included features. (Note: `pd_estimate` is also a constructed ECL input per A02 §3.3 — exclusion here is from the predictor feature matrix only.) |
| `delinquency_state` | EXCLUDE as predictor | Deterministic transform of the target; would produce tautological discrimination. |
| `default_flag` | EXCLUDE | Target transform; excluded from the feature matrix per A02 §3.4. |

### 1.2 — Inputs (enumerated as verifiable from A01–A03)

The following features are named as PD/LGD/EAD inputs in A02 §3.2. Each is enumerated here with its role. Verification against the full `feature_inventory.csv` is required before this list can be treated as the complete included-feature set.

| Feature | Role (per A02 §3.2) | Status |
| :--- | :--- | :--- |
| `FICO_BOR` | PD input; LGD covariate | `[VERIFY: feature_inventory.csv]` |
| `ORG_LTV` | PD input; LGD covariate | `[VERIFY: feature_inventory.csv]` |
| `DTI` | PD input | `[VERIFY: feature_inventory.csv]` |
| `AGE` | PD input; LGD covariate | `[VERIFY: feature_inventory.csv]` |
| `ORG_UPB` | EAD input | `[VERIFY: feature_inventory.csv]` |
| `ORG_CLTV` | LGD covariate | `[VERIFY: feature_inventory.csv]` |
| `PROP_TYPE` | LGD covariate | `[VERIFY: feature_inventory.csv]` |
| `OCCU_STAT` | Possible signal | `[VERIFY: feature_inventory.csv]` |

> **Coverage note:** This is not asserted as the complete included-feature list. The complete list is the set of `INCLUDE` decisions in `feature_inventory.csv` (per A03 §6.1). This section enumerates only what is verifiable from A01–A03; any feature in `feature_inventory.csv` marked `INCLUDE` that does not appear above must be added here by the analyst.

**Inclusion rule (planning):** A feature is included if and only if (i) `feature_inventory.csv` records an `INCLUDE` decision, (ii) it does not match `KNOWN_LEAKAGE` (A03 §7.1), and (iii) it passes the NB01 Phase 5 leakage scan without an unresolved flag routed to A04.

**Status:** `Decided` for the named exclusions in §1.1; `Deferred to feature_inventory.csv / NB01 — owner: [ANALYST: role]` for the complete included-feature enumeration in §1.2.

---

## 2. How is each feature transformed?

> **Planning note:** This section records the intended transform for each feature. Where a transform is not yet decided, the decision is deferred to the named artifact/owner.

| Feature | Intended transform | Status |
| :--- | :--- | :--- |
| `FICO_BOR` | `[ANALYST: intended transform — e.g., passthrough / binning / capped linear]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `ORG_LTV` | `[ANALYST: intended transform]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `DTI` | `[ANALYST: intended transform]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `AGE` | `[ANALYST: intended transform]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `ORG_UPB` | `[ANALYST: intended transform]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `ORG_CLTV` | `[ANALYST: intended transform]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `PROP_TYPE` | `[ANALYST: intended encoding]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |
| `OCCU_STAT` | `[ANALYST: intended encoding]` | `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]` |

> **Constructed parameters** (`pd_estimate`, `lgd_final`, `ead_final`) are specified in A02 §3.3 and are not duplicated here as feature transforms. Their construction is recorded in A02/A05.

**Status:** `Deferred to A02 §3.3 / NB03 — owner: [ANALYST: role]`

---

## 3. Which features are proxies, and what gap do they introduce?

| Proxy | Intended concept | Gap introduced | Mitigation |
| :--- | :--- | :--- | :--- |
| `default` as `90+ DPD` (as recorded in the A04 feature block) | A credit event resulting in loss | Forbearance, modification, and short sale are not captured | Documented as a limitation; industry standard. **Target framing is owned by A01 §5 / A02 §1 — see deferral note below.** |
| `delinquency_state` (Stage 3 driver: `default × 5` / `>= 3`) | IFRS 9 Stage 3 / credit-impaired proxy | Stage 3 is a construction, not an observed IFRS 9 Stage 3 (C6 limitation) | Documented as `C6` / `L2` in `FM-CFR-001`. |
| Top-decile SICR proxy (Stage 2 driver) | IFRS 9 Stage 2 SICR | Stage 2 is SICR-by-construction, not observed credit deterioration (`L2`) | Documented as `L2`; mechanism in A05 §5.2. |

> **Deferral note on the target-proxy threshold:** The A04 feature block recorded `90+ DPD`; A01 §5 and A02 §1 record a `16+` day framing. A04 does not restate the threshold — the target framing is **owned by A01/A02**. A04 records only that a proxy exists and what gap it introduces. The threshold itself should be read from A01 §5.

**Status:** `Decided` for the proxy list and gaps above; `Deferred to A01 §5 / A02 §1 — owner: [ANALYST: role]` for the target-proxy threshold itself.

---

## 4. Why does each feature matter (business meaning)?

> **Planning note:** This section records the intended business meaning per feature. Where the meaning is not yet decided, it is deferred to the named artifact/owner.

| Feature | Business meaning (intended) | Status |
| :--- | :--- | :--- |
| `FICO_BOR` | Borrower creditworthiness at origination; primary PD driver | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `ORG_LTV` | Leverage at origination; PD driver and LGD covariate (collateral cushion) | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `DTI` | Borrower repayment burden; PD driver | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `AGE` | Loan seasoning; PD driver and LGD covariate | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `ORG_UPB` | Exposure at origination; EAD input | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `ORG_CLTV` | Combined leverage; LGD covariate | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `PROP_TYPE` | Collateral type; LGD covariate | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |
| `OCCU_STAT` | Occupancy status; possible risk signal | `Deferred to A02 / NB02 — owner: [ANALYST: role]` |

**Status:** `Deferred to A02 / NB02 — owner: [ANALYST: role]`

---

## 5. How stable is each feature?

> **Planning note:** This section plans the stability evidence to be produced. It does not record measured stability (that belongs to A11 / execution artifacts). The LGD-construction paragraph that previously sat in this slot has been moved to A05 (Methodology Document) per the analyst's decision — LGD construction is a methodology topic, not a feature-stability topic.

**Planned stability evidence:**

| Evidence | What it will show | Planned location | Status |
| :--- | :--- | :--- | :--- |
| Feature distributions by vintage | Whether feature mix shifts across origination years | NB02 (EDA) | `Deferred to NB02 — owner: [ANALYST: role]` |
| Feature distributions by segment (e.g., FICO band, LTV band) | Whether feature behavior is stable across risk segments | NB02 (EDA) | `Deferred to NB02 — owner: [ANALYST: role]` |
| Temporal stability of feature–target relationship (where observable) | Whether the univariate signal is stable over time | NB02 / NB03 | `Deferred to NB02/NB03 — owner: [ANALYST: role]` |
| Populated-feature stability set | The exact features for which stability is assessed | NB02 | `Open — decision needed by NB02 kickoff` |

> **Moved content note:** LGD construction (base 0.45 + age/LTV/property adjustments, clipped `[0.10, 0.90]`) is recorded in **A05 (Methodology Document)**, not here. The earlier note that LGD is a function of PD and not empirically calibrated, and that its mean is a construction artifact, travels with that paragraph to A05.

**Status:** `Deferred to NB02 — owner: [ANALYST: role]` for the planned evidence; `Open — decision needed by NB02 kickoff` for the populated-feature stability set.

---

## 6. What fairness implications does each feature carry?

> **Planning note:** All fairness testing in this framework is proxy-based; the Fannie Mae public data contains no direct protected-class attributes (A03 §10.1). Geographic proxies are not protected classes. Per-feature fairness implications are enumerated below; the testing plan for each is deferred to the named artifact/owner.

| Feature | Fairness implication (intended) | Testing plan | Status |
| :--- | :--- | :--- | :--- |
| `FICO_BOR` | Correlates with race/ethnicity due to historical inequities; indirect disparate impact risk | Fair-lending testing required; proxy-based | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `ORG_LTV` | May correlate with geography and borrower demographics; indirect disparate impact risk | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `DTI` | May correlate with borrower demographics; indirect disparate impact risk | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `AGE` | Seasoning-linked; may correlate with vintage/geography | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `ORG_UPB` | Loan-size-linked; may correlate with geography | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `ORG_CLTV` | Combined leverage; may correlate with geography | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `PROP_TYPE` | Property-type-linked; may correlate with geography/borrower demographics | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |
| `OCCU_STAT` | Occupancy-linked; may correlate with borrower demographics | Proxy-based disparate-impact check | `Deferred to A06 / NB04 — owner: [ANALYST: role]` |

> **Scope caveat:** No direct protected-class testing is possible on this dataset (`L7` in `FM-CFR-001`). Geographic proxies (`PROP_STATE`, `MSA`, `ZIP`) are used solely as exploratory fairness proxies (A03 §10.1) and are not treated as protected classes.

**Status:** `Decided` that all fairness testing is proxy-based and per-feature; `Deferred to A06 / NB04 — owner: [ANALYST: role]` for the per-feature testing plan execution.

---

## 7. Deferral and open-item register (A04 scope)

| # | Item | Deferred to | Owner | Trigger/milestone |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Complete included-feature enumeration | `feature_inventory.csv` / NB01 | `[ANALYST: role]` | NB01 Phase 6 completion |
| 2 | Per-feature transforms | A02 §3.3 / NB03 | `[ANALYST: role]` | NB03 specification |
| 3 | Target-proxy threshold | A01 §5 / A02 §1 | `[ANALYST: role]` | A01/A02 finalization |
| 4 | Per-feature business meaning | A02 / NB02 | `[ANALYST: role]` | NB02 EDA |
| 5 | Per-feature stability evidence | NB02 | `[ANALYST: role]` | NB02 kickoff |
| 6 | Populated-feature stability set | NB02 | `[ANALYST: role]` | NB02 kickoff |
| 7 | Per-feature fairness testing plan | A06 / NB04 | `[ANALYST: role]` | A06 / NB04 |

---

## 8. Document control

| Field | Value |
| :--- | :--- |
| **Document ID** | `[ANALYST: Document ID — cross-referenced elsewhere as FM-FDL-001]` |
| **Version** | `0.1.0-draft` |
| **Status** | Draft — pending analyst insertion of `[ANALYST: ...]` values and `[VERIFY: ...]` confirmations |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | — (new artifact) |
| **Related documents** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03), A05 (Methodology Document), `FM-DCR-001` (A07), `FM-CFR-001`, `feature_inventory.csv` |
| **Next review** | Upon completion of NB01 Phase 6 (`feature_inventory.csv`) or next version bump |
