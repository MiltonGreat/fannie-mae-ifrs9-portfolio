# Fannie Mae IFRS 9 ECL Demonstration — Consolidated Findings Register

**Document ID:** FM-CFR-001  
**Version / Status:** 1.4 / Frozen at NB03 v1.3 execution (run_id `20261021_121550`)  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Consolidated Findings Register (CFR)  
**Source registers:** `outputs/reports/notebook_03_limitations.csv`, the NB03 findings register (L1–L10), and the NB01 findings  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan) · FM-AR-001 (Approval Record)

---

## At a glance

| Field | Value |
|---|---|
| **Total findings** | 13 (3 from NB01, 10 from NB03) |
| **By severity** | 0 CRITICAL · 5 HIGH · 5 MEDIUM · 3 LOW |
| **By status** | 13 DOCUMENTED · 0 OPEN · 0 DEFERRED |
| **Notebooks producing findings** | NB01 (data limitations), NB03 (methodology limitations) |
| **Notebooks producing no findings** | NB02 (EDA), NB04 (quantifies), NB05 (maps), NB06 (exports) |
| **The register's key claim** | The methodology has **zero critical defects**; its limitations are documented, quantified, and mapped to regulatory requirements |

> **Document purpose.** This register consolidates every finding produced across NB01, NB02, and NB03 into a single document. It is the complete record of what was found — not a summary, not a re-derivation.
>
> **The critical difference from the Lending Club CFR.** The Fannie Mae pipeline has **0 CRITICAL findings** and **0 OPEN findings**. The methodology's findings are all **documented limitations** (L1–L10), not defects requiring remediation. **The absence of CRITICAL findings is not evidence of a better model** — it is evidence of a **different validation paradigm**. The Lending Club project validates a *model that can be remediated*; the Fannie Mae project validates a *methodology whose limitations are disclosed and scoped*. A reader who compares the two CFRs should see the paradigm difference, not a quality difference.

---

## 1. Purpose and scope of the register

This register consolidates every finding produced across NB01, NB02, and NB03 into a single document. It is the complete record of what was found — not a summary, not a re-derivation.

**The critical difference from Lending Club.** The Fannie Mae pipeline has **0 CRITICAL findings** and **0 HIGH findings in the traditional "defect" sense**. The methodology's findings are all **limitations** (L1–L10), and the register's job is to make those limitations explicit, quantified, and assigned. **The absence of CRITICAL findings is not a sign of a better model** — it is a sign of a different validation paradigm.

**On the distinction from the DCR.** The DCR (FM-DCR-001) defines the criteria and reports the measured results against them. This register records the *findings* — the limitations, their severity, their status, and their cross-references. **The DCR is the standard and the results; the CFR is the findings.** A finding can appear in both (a limitation with an associated guard result is both a DCR entry and a CFR entry), but the two documents serve different readers: the DCR's reader asks "did the criteria pass?"; the CFR's reader asks "what are the limitations?"

**On the distinction from the framework document.** The framework document (MAP-STF-FM-001) interprets the findings for a reader who wants the methodology's assessment. The CFR records the findings; the framework document assesses them. **The CFR is the evidence; the framework document is the interpretation.**

---

## 2. Register structure and schema

The consolidated register has 12 columns per finding:

| Column | Type | Purpose |
|---|---|---|
| `Limitation_ID` | string | L1–L10 for methodology limitations; L-01.XX for NB01 data limitations |
| `Source_Notebook` | string | Which notebook produced it |
| `Severity` | enum | HIGH / MEDIUM / LOW — **no CRITICAL** for limitations |
| `Finding` | string | Short name |
| `Description` | string | What the limitation is |
| `Evidence_Reference` | string | Where the evidence is |
| `Impact` | string | What it means for the ECL |
| `Mitigation` | string | What has been done or should be done |
| `Owner` | string | Who owns the mitigation |
| `Timeline` | string | When it is due |
| `Status` | enum | DOCUMENTED / OPEN / DEFERRED |
| `Reg_Reference` | string | IFRS 9 / Basel III / OSFI E-23 reference |

**The schema has two important differences from the Lending Club CFR:**

1. **No CRITICAL severity.** The Fannie Mae findings are all **limitations**, not defects. A limitation can be HIGH but not CRITICAL — CRITICAL would mean the methodology is fundamentally flawed, which it is not. **The absence of CRITICAL is a deliberate schema choice**, not an oversight: a methodology demonstration is not a deployable model, and its "worst" finding is a scope limitation, not a defect that blocks deployment.

2. **Status uses DOCUMENTED instead of RESOLVED.** A limitation is not "resolved" — it is documented and mitigated. **The status change reflects the different nature of the finding**: a Lending Club finding is a defect to be closed; a Fannie Mae finding is a limitation to be disclosed.

**On the "no CRITICAL" claim.** This is not a claim that the methodology is perfect. It is a claim that **the methodology's limitations are all of a kind that can be disclosed, not a kind that invalidates the methodology**. A reader who reads "0 CRITICAL" and concludes "the methodology is flawless" has misread the register. The HIGH findings (L1, L2, L6, L7, L8) are substantive limitations; they are HIGH because they matter, and they are not CRITICAL because they are disclosed, quantified, and scoped.

---

## 3. Findings by notebook

### 3.1 — NB01 (Data Preparation) — 3 findings

| ID | Severity | Finding | Status |
|---|---|---|---|
| L-01.01 | LOW | Sample size 100K (from ~1M full dataset) | DOCUMENTED |
| L-01.02 | LOW | U.S. data only (not Canadian) | DOCUMENTED |
| L-01.03 | LOW | `pd_estimate` constructed, not fitted | DOCUMENTED |

**The NB01 findings are data-preparation limitations.** The sample size is a documented scope limitation (the 100K sample is representative but not exhaustive). The U.S. data limitation is the jurisdictional issue. The constructed `pd_estimate` is the most important — it is the methodology's core assumption, and it is documented in FM-FDL-001 §5.2.

**On the severity of the NB01 findings.** The v1.3 register listed L-01.01 as HIGH, L-01.02 as MEDIUM, L-01.03 as MEDIUM. The v1.4 register lists all three as LOW. **The severity is a judgment call.** The v1.4 severity reflects that these are **scope limitations** (the sample size is a scope choice, the U.S. data is a scope choice, the constructed `pd_estimate` is a methodology choice), not **defects**. A scope limitation is LOW because it does not require remediation; it requires disclosure. If you want to keep the v1.3 severities (HIGH/MEDIUM/MEDIUM), the register still works — the severity column is a labeling choice, and the "why" is in the description. **My recommendation is LOW for all three**, because they are consistently scope limitations, and mixing severities for scope limitations suggests some are more remediable than others. Your call.

### 3.2 — NB02 (EDA) — 0 findings

**NB02 produces no findings.** It is descriptive only — it reports what is in the data, not whether it is good or bad. This is by design. The EDA notebook is not a validation stage; it is an exploration stage.

**The absence of NB02 findings is itself informative.** It means the EDA did not surface any new issues — the data quality limitations were already caught in NB01. A cleaner data preparation process produces fewer downstream findings.

### 3.3 — NB03 (Calibration) — 10 limitations (L1–L10)

| ID | Severity | Finding | Status | Reg Reference |
|---|---|---|---|---|
| L1 | HIGH | PD-LGD correlation is constructed | DOCUMENTED | IFRS 9 §5.5.3 |
| L2 | HIGH | SICR uses top-decile proxy | DOCUMENTED | IFRS 9 §5.5.9 |
| L3 | MEDIUM | LGD is a function of PD | DOCUMENTED | IFRS 9 §5.5.3 |
| L4 | MEDIUM | Prepayment/curtailment are assumptions | DOCUMENTED | IFRS 9 §5.5.3 |
| L5 | MEDIUM | EIR is industry average (3.5%) | DOCUMENTED | IFRS 9 §5.5.3 |
| L6 | HIGH | PD cap (20%) masks tail convexity | DOCUMENTED | IFRS 9 §5.5.17 |
| L7 | HIGH | Fairness testing incomplete | DOCUMENTED | OSFI E-23 §6.0 |
| L8 | HIGH | U.S. data as Canadian proxy | DOCUMENTED | OSFI E-23 applicability |
| L9 | MEDIUM | Scenario weights and multipliers are assumptions | DOCUMENTED | IFRS 9 §5.5.18 |
| L10 | MEDIUM | Capital coverage is a diagnostic | DOCUMENTED | Basel III |

**Summary:** 0 CRITICAL, 5 HIGH, 5 MEDIUM. All 10 DOCUMENTED.

**The NB03 limitations are the methodology's honest scope.** Every limitation is documented in the `notebook_03_limitations.csv` and cross-referenced in the framework document (MAP-STF-FM-001). The **DOCUMENTED** status means the limitation is explicit, not hidden — which is the methodology's defensibility claim.

**On the HIGH findings.** L1 (PD-LGD correlation is constructed), L2 (SICR proxy), L6 (PD cap), L7 (fairness not tested), L8 (U.S. data) are the substantive limitations. **A reader who reads only the HIGH findings gets the full picture of the methodology's defensibility.** The MEDIUM findings (L3, L4, L5, L9, L10) are parameter assumptions and framing choices; they matter for interpretation but do not change the methodology's paradigm.

### 3.4 — NB04, NB05, NB06 — 0 findings each

**NB04 produces no findings.** It consumes the NB03 limitations and **quantifies** them (e.g., the PD-cap understatement — the `cap_effect_pct` in NB04's results). NB04 is where the limitations become numbers, not where new limitations are found.

**NB05 produces no findings.** It consumes the NB03 limitations and **maps** them to regulatory requirements. NB05 is where the limitations become compliance statements.

**NB06 produces no findings.** It exports the results as dashboard artifacts. NB06 is terminal — no downstream analysis, no findings.

**On the "quantifies, maps, exports" distinction.** These are not passive roles. NB04's quantification (e.g., "the PD cap understates tail ECL by X%") is a *contribution* to the methodology's defensibility — a limitation that is quantified is more defensible than a limitation that is merely named. The register's job is to record the limitation; the downstream notebooks' job is to make the limitation *material*. The distinction is why the register has a "Source Notebook" column that names the *origin* notebook (NB01, NB03) rather than the notebook that carried it forward (NB04, NB05).

---

## 4. Findings by severity

| Severity | Count | Findings |
|---|---|---|
| CRITICAL | 0 | — |
| HIGH | 5 | L1, L2, L6, L7, L8 |
| MEDIUM | 5 | L3, L4, L5, L9, L10 |
| LOW | 3 | L-01.01, L-01.02, L-01.03 |

**The absence of CRITICAL findings is the register's most important feature.** It means the methodology is fundamentally sound — it has limitations, but no defects that invalidate the approach. This is the difference between *"the methodology is wrong"* and *"the methodology has a documented scope."*

**On the HIGH findings.** L1 (PD-LGD correlation is constructed), L2 (SICR proxy), L6 (PD cap), L7 (fairness not tested), L8 (U.S. data) — these are the substantive limitations. A validator who reads only the HIGH findings gets the full picture of the methodology's defensibility. **The HIGH findings are not defects** — they are limitations that a production implementation would need to address, but that the demonstration discloses.

**On the LOW findings (NB01).** The v1.3 register assigned HIGH/MEDIUM/MEDIUM severities to L-01.01, L-01.02, L-01.03. The v1.4 register assigns LOW to all three. See §3.1 for the rationale — they are scope limitations, not defects. **If you want to keep the v1.3 severities, the register still works**, but the inconsistency (a "HIGH" finding that is a scope limitation) is worth reconsidering. Your call.

---

## 5. Findings by status

| Status | Count | Notes |
|---|---|---|
| DOCUMENTED | 13 | All limitations are documented |
| OPEN | 0 | No open findings — the methodology is complete |
| DEFERRED | 0 | No deferred findings |

**The "DOCUMENTED" status is the register's second-most important feature.** It means every limitation has been explicitly acknowledged in the notebook, the findings register, and the framework document. **There are no hidden limitations.**

**This is the opposite of the Lending Club CFR.** Lending Club has 21 OPEN findings (the model needs remediation). Fannie Mae has 0 OPEN findings (the methodology is complete, with documented limitations). **The difference is the paradigm:** a model needs to be fixed; a methodology needs to be documented.

**On the "DOCUMENTED" vs "RESOLVED" distinction.** A "RESOLVED" finding in Lending Club means the issue has been fixed (e.g., leakage removed). A "DOCUMENTED" finding in Fannie Mae means the limitation has been disclosed and accepted (e.g., PD cap masks tail convexity). **The two statuses are not the same kind of thing** — one is a closure state, the other is a disclosure state. The CFR's schema reflects this.

**On the absence of OPEN findings.** This does not mean there is nothing to do. It means the methodology's *open items* are tracked in other documents: the framework document's observations (MAP-STF-FM-001 §7), the Remediation Plan (MAP-RP-001 · FM-R-06 through FM-R-10), and the DCR's open items (FM-DCR-001 §9.1). **The CFR's "0 OPEN" is a claim about the register's contents, not a claim about the project's open work.** A reader who sees "0 OPEN" and concludes "the project is done" has misread the register — the open work is elsewhere.

---

## 6. Cross-register reconciliation

The Fannie Mae register has **no cross-register chain in the same sense as Lending Club.** The limitations are carried forward from NB03 to NB04 to NB05, but they are not re-found at each stage. **The register's reconciliation is a "quantify → map → export" chain**, not a "find → re-find" chain.

| Limitation | NB03 | NB04 | NB05 | NB06 |
|---|---|---|---|---|
| L1 (PD-LGD) | Documented | Quantified | Mapped | Exported |
| L2 (SICR) | Documented | Quantified | Mapped | Exported |
| L3 (LGD) | Documented | Quantified | Mapped | Exported |
| L4 (Prepayment) | Documented | Quantified | Mapped | Exported |
| L5 (EIR) | Documented | Quantified | Mapped | Exported |
| L6 (PD cap) | Documented | **Quantified (pre-cap understatement)** | Mapped | Exported |
| L7 (Fairness) | Documented | **Scoped (not tested)** | Mapped | Exported |
| L8 (U.S. data) | Documented | Scoped | Mapped | Exported |
| L9 (Scenario) | Documented | Quantified | Mapped | Exported |
| L10 (Capital) | Documented | **Reframed** | Mapped | Exported |

**The NB04 column is the most important** — it is where the limitations become *material*. L6 (PD cap) becomes "the tail ECL is understated by X%" (the `cap_effect_pct` in NB04's results); L7 (fairness) becomes "not tested, and here is why" (FM-MD-001 §6.1); L10 (capital) becomes "ECL ratio of assets, not CET1" (the reframe in NB04's coverage section). **The limitations are not just documented — they are made measurable.**

**On the "not re-found" distinction.** In Lending Club, a finding in NB03 might be a re-finding of an NB02 finding (e.g., F-03.04 is a re-finding of F-02.04, both about XGBoost's poor calibration). **In Fannie Mae, the limitations are not re-found** — NB04 quantifies L6, but it does not re-raise it as a new finding. The reason is the paradigm: Lending Club's validator *independently confirms* the developer's findings; Fannie Mae's downstream notebooks *make the limitations material*. **The two are different validation relationships**, and the CFR's reconciliation table reflects the difference.

---

## 7. Open items and dependencies

### 7.1 — Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **L6 — PD cap** | NB04 quantifies the pre-cap understatement. A future version could remove the cap (with a tail-risk analysis) or justify 20% explicitly. | MAP-RP-001 · FM-R-10 (**MAJOR** version bump if changed); FM-DCR-001 §9.1 |
| 2 | **L7 — fairness gap** | Fairness is not tested. A production implementation would need HMDA-equivalent data. | MAP-RP-001 · FM-R-09 (longer-horizon); FM-DCR-001 §9.1 |
| 3 | **L2 — SICR proxy** | The top-decile proxy is a documented limitation. A production implementation would need current credit characteristics. | MAP-RP-001 · FM-R-07; FM-DCR-001 §9.1 |
| 4 | **L9 — scenario assumptions** | The 45/30/15/10 weights and the 1.0–4.5× multipliers are analyst-specified, not calibrated. A future version could calibrate against 2008 actuals. | MAP-RP-001 · FM-R-06 (**MAJOR** version bump if changed) |

**On the routing column.** Each open item is routed to a **MAP-RP-001 finding** (which has an owner, timeline, and closure evidence) and — where applicable — to a **DCR open item** (which is a proposed threshold change under the DCR's own change-control chain). **The CFR does not have its own open items** — the limitations are closed by the framework document's observations, not by a CFR-specific process. **The CFR's "0 OPEN" status is therefore not the whole story** — the open work is in the Remediation Plan, the DCR, and the framework document.

**On the v1.3 register's "No open findings" claim.** The v1.3 register said "No open findings. The methodology is complete with documented limitations." That claim is correct as a statement about the CFR's contents, but it is misleading as a statement about the project. **The v1.4 register clarifies** — the CFR's open items are routed to their actual tracking locations, and the register's "0 OPEN" is explicitly framed as "about the register, not the project."

### 7.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | NB01 data limitations, NB03 methodology limitations |
| Downstream | NB04 (quantifies), NB05 (maps to regulatory), NB06 (exports), MAP-STF-FM-001 (interprets), FM-AR-001 (decides) |
| Reproducibility | The limitations CSV's hash is recorded in the NB03 audit log |
| Cross-project | The Fannie Mae findings are **all limitations** (0 CRITICAL); the Lending Club findings include **CRITICAL defects** (leakage, fairness). See §1 for the paradigm distinction |

---

*End of document.*
