# Problem Definition Record — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-PDR-001  
**Version / Status:** 0.1 / Draft — retrospective  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Problem Definition Record (PDR)  
**Companion documents:** FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

> **Retrospective framing.** This document was reconstructed from the FM pipeline's implementation, not written before it. The pipeline's notebooks, outputs, and the FDL/DCR/CFR record what the problem definition *turned out to be*. A PDR written before the analysis would have stated the problem definition *in advance*, and would have been the document that constrained the subsequent work. This version is the retrospective: it describes the problem definition the pipeline implies. **Where this document and the FDL/DCR/CFR disagree, the FDL/DCR/CFR take precedence** — they were revised against the notebook output, while this document is a reconstruction.

---

## 1. Purpose and scope

This PDR records the problem the Fannie Mae IFRS 9 ECL framework was built to address: what decision the framework supports, what population it applies to, what the target is, and what constraints the analysis operated under.

**This is not a validation opinion.** The framework has not been independently validated — it is a methodology demonstration. This document records the problem the demonstration addresses, not the quality of the demonstration.

---

## 2. The decision supported

The pipeline supports **a methodology demonstration, not a business decision.** The framework does not decide whether to provision, whether to lend, or whether to deploy. It demonstrates that a pipeline **can** implement the IFRS 9 ECL arithmetic on a public dataset, with documented assumptions.

**What the pipeline supports:**
- A methodology demonstration for a portfolio, technical, or regulatory audience.
- The pipeline's outputs (PD, LGD, EAD, staging, probability-weighted ECL) as evidence that the arithmetic works.

**What the pipeline does not support:**
- A production ECL estimate.
- A provisioning decision.
- A capital adequacy determination.
- A regulatory compliance claim.

**The distinction is the framework's central claim** (see MAP-STF-FM-001 §1 and the FM README's "What this framework does NOT demonstrate" section). The PDR records the distinction at the problem-definition stage, so a reader who starts here knows what the pipeline is and is not for.

`[ANALYST: insert the analyst's framing of the decision the pipeline supports. The "what the pipeline supports" list above is derived from the framework document and the README; the analyst's framing is the paragraph that says what a reader should do with the pipeline's output. If the analyst's framing is "use this as a teaching example," say so; if it is "use this as a starting point for a production build," say so; if it is something else, say that.]`

---

## 3. Population

The pipeline operates on a **100,000-row random sample** of the Fannie Mae Single-Family Loan Performance dataset.

| Field | Value |
|---|---|
| **Source population** | Fannie Mae Single-Family Loan Performance (public tape; `train4.csv` from `Fannie Mae.zip`) |
| **Full population size** | 1,672,822 rows |
| **Sample size** | 100,000 rows |
| **Sample method** | Random sample, seed 42 |
| **Sample default rate** | 0.94–0.95% |
| **Post-filter population** | 100,000 rows (no rows excluded by the target definition) |

**On the population's scope.** The population is Fannie Mae's loan tapes — U.S. single-family mortgages. It does not include Canadian portfolios, commercial mortgages, or any other loan type. The pipeline's conclusions are illustrative for a Canadian bank's portfolio, not transferable (L8 in FM-CFR-001).

**On the sampling.** The random sample is not stratified. The sample's default rate matches the population's default rate to two decimal places, which is the sample-quality check. A stratified sample would be more efficient for some purposes; a random sample is the conservative choice for a methodology demonstration.

`[ANALYST: insert the population's business context. Why Fannie Mae? What does the dataset represent in the mortgage market? What vintage coverage does it have? The notebook output does not contain this context; it is the analyst's framing of why this dataset is the right one for the demonstration.]`

---

## 4. Target definition

| Field | Value |
|---|---|
| **Target** | `default` (binary) |
| **Definition** | `1` if the loan status is a "bad" status (Charged Off, Late (31–120 days), In Grace Period, Late (16–30 days)); `0` if the loan status is a "good" status (Fully Paid, Current) |
| **Threshold** | 16+ days past due |
| **Exclusions** | No rows excluded (all loans fall into a good or bad status) |

**The threshold is 16+ days, not IFRS 9's 30+ days.** This is a deliberate design decision: the target definition is **for the ECL demonstration's purposes**, not for IFRS 9 provisioning. The 16+ day threshold produces a higher default rate than a 30+ day threshold would, which makes the demonstration's numbers larger and therefore easier to inspect. **The framework does not claim to be IFRS 9-compliant because of this threshold** (see L8 in FM-CFR-001; the framework's README "What this framework does NOT demonstrate" section).

`[ANALYST: insert the reasoning for the 16+ day threshold. Was it chosen because the Fannie Mae tape's `loan_status` field has those specific categories? Was it chosen to match a benchmark? Was it chosen to produce a target rate in a particular range? If the answer is "it was chosen because the status categories are what they are," state that — it is the honest answer.]`

---

## 5. Constraints

The pipeline operated under the following constraints:

| Constraint | What it means |
|---|---|
| **Public dataset** | Fannie Mae Single-Family Loan Performance; no proprietary bank data |
| **U.S. data only** | No Canadian portfolio validation (L8) |
| **No current credit characteristics** | SICR uses a top-decile proxy, not an observed measurement (L2) |
| **No direct protected attributes** | Fairness testing is geographic-proxy only (L7) |
| **No macro indicators** | Scenario multipliers are analyst-specified, not calibrated to macro variables (L9) |
| **Methodology demonstration, not production** | Parameters are specified, not fitted; ECL is illustrative, not a forecast |

**The constraints are not limitations of the pipeline's engineering** — they are limitations of the data. The pipeline implements the methodology the data supports. A production implementation would need data the pipeline does not have.

`[ANALYST: insert the analyst's framing of the constraints. The list above is the factual constraints; the framing is why they matter for the demonstration's claims. The "why it matters" is the analyst's voice: a constraint that is named but not interpreted is a fact without a consequence.]`

---

## 6. Success criteria

The pipeline's success criteria are the criteria the DCR (FM-DCR-001) defines, applied at the methodology level:

| Criterion | Threshold | Source |
|---|---|---|
| AP-02 (LGD band) | 40%–60% target; 30%–70% failure | DCR §3.3 |
| AP-03 (EAD ratio) | 60%–75% target; 50%–85% failure | DCR §3.3 |
| AP-04 (ECL ratio of EAD) | 1%–3% target; 0.5%–5% failure | DCR §3.3 |

**The pipeline's success is not "all guards PASS."** The pipeline's success is that the guards **fire as designed** — including the guards that fire because the data has a known property (AP-03 fires because the portfolio is young; that is the guard working correctly, not the pipeline failing).

**In this run:** AP-02 PASS, AP-03 OUTSIDE FAILURE (expected), AP-04 PASS. The pipeline is **successful as a demonstration** — the guards fire as designed, and the one exception is documented.

`[ANALYST: insert the analyst's framing of "success" for the pipeline. The DCR defines the guards; the analyst defines what "success" means for the demonstration. Is it "the guards fired correctly"? Is it "the numbers are in a defensible range"? Is it "the methodology's limitations are documented"? All three are true; the analyst's framing is what a reader should take away.]`

---

## 7. What the pipeline does not decide

**The pipeline does not decide:**
- Whether the framework is approved.
- Whether the methodology is production-ready.
- Whether the ECL estimate is correct.
- Whether the framework is IFRS 9-compliant.

**These decisions belong to the analyst**, in the Approval Record (FM-AR-001) and any Validation Opinion that might be written about the framework. **The pipeline's job is to compute; the analyst's job is to decide.** The PDR records the boundary because a reader who starts with the PDR should know where the pipeline's authority ends.

`[ANALYST: insert the analyst's framing of the boundary. The paragraph above states the boundary; the framing is why it matters — what would go wrong if a reader crossed it? The answer is different for each decision. "Whether the framework is approved" is a governance failure if crossed; "whether the ECL estimate is correct" is a validation failure; "whether the framework is IFRS 9-compliant" is a regulatory failure. The analyst's framing names the failure modes.]`

---

## 8. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-PDR-001 |
| **Version** | 0.1 |
| **Status** | Draft — retrospective; pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | At the next MAJOR version bump of the framework |

---

## 9. What this document contains, and what it does not

### What it contains (from the notebook output and the FM documents)

- The decision the pipeline supports (a methodology demonstration).
- The population (Fannie Mae sample, 100,000 rows).
- The target definition (16+ days past due; binary).
- The constraints (public data, U.S. only, no current credit, no protected attributes, no macro).
- The success criteria (the DCR's guard bands).
- The boundary between the pipeline's authority and the analyst's authority.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§2** — the analyst's framing of the decision the pipeline supports.
- **§3** — the population's business context (why Fannie Mae, what vintage coverage).
- **§4** — the reasoning for the 16+ day threshold.
- **§5** — the analyst's interpretation of the constraints' consequences.
- **§6** — the analyst's framing of "success."
- **§7** — the analyst's framing of why the boundary matters.
- **§8** — the document owner's name and role.

**The document's structure is retrospective.** The factual content (population, target, constraints, success criteria) is sourced from the notebook output and the FM documents. The analytical content (why Fannie Mae, why 16+ days, why the boundary matters) is the analyst's contribution. **A PDR written before the analysis would have had the analytical content and would have derived the factual content from it. This retrospective PDR has the factual content and marks the analytical content as the analyst's contribution.** Both are useful; the retrospective version is the one this project has, and this document is honest about the fact.
