# Fannie Mae IFRS 9 ECL Demonstration — Approval Record

**Document ID:** FM-AR-001  
**Version / Status:** 1.4 / Draft pending Methodology Owner sign-off  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Approval Record (AR)  
**Decision-maker:** Methodology Owner  
**Frozen at:** NB03 v1.3 execution (run_id `20261021_121550`)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 (IFRS 9 Stress Testing Framework) · MAP-RP-001 (Remediation Plan)

---

## Record status at a glance

| Field | Value |
|---|---|
| **Record state** | DRAFT — awaiting Methodology Owner signature |
| **Framework document status** | ACCEPTED as a methodology demonstration (recommendation) |
| **MAP Validator's position** | ACCEPT (recommendation to the Methodology Owner) |
| **Methodology Owner's decision** | `[ANALYST: decision — ACCEPT / CONDITIONAL / REJECT]` |
| **When decision is signed** | Record becomes binding; the methodology may be cited as accepted |
| **Next review** | At next parameter version bump (no calendar schedule) |

> **Document purpose.** This record documents the formal sign-off for the Fannie Mae IFRS 9 ECL Demonstration. **The methodology is not being "approved for deployment" — it is being approved as a demonstration of IFRS 9 methodology.**
>
> **On the record's state.** This record is a **draft** until the Methodology Owner's signature is applied. The MAP Validator's ACCEPT is a *recommendation*, not a *decision* by the Methodology Owner. The record becomes binding when the signature block (§6) is complete.
>
> **On the meaning of "approval" in this record.** Unlike the Lending Club Approval Record, this record is not about *deployment authorization* — it is about *methodology acceptance*. The Methodology Owner is not authorizing deployment; they are accepting the methodology as a demonstration of IFRS 9 concepts. **The decision's meaning is narrower, and the record's language reflects that.**

---

## 1. Purpose and scope of the record

This record documents the formal sign-off for the Fannie Mae IFRS 9 ECL Demonstration. **The methodology is not being "approved for deployment" — it is being approved as a demonstration of IFRS 9 methodology.**

**The distinction from Lending Club.** Lending Club's Approval Record is about **deployment authorization** — the MRC Chair authorizes or rejects deployment of a model. Fannie Mae's Approval Record is about **methodology acceptance** — the Methodology Owner accepts the methodology as a demonstration. The record's language is different, and the decision's meaning is different.

**On the difference between the Lending Club and Fannie Mae approval paradigms.** A reader who compares the two Approval Records should see the difference, not the same document with different names. Lending Club's approval is a **deployment gate**; Fannie Mae's approval is a **methodology endorsement**. The Lending Club record's decision is `APPROVE REJECT / CONDITIONAL / OVERRIDE` (a deployment decision); the Fannie Mae record's decision is `ACCEPT / CONDITIONAL / REJECT` (an endorsement decision). The two are not interchangeable.

---

## 2. Record structure and schema

| Section | Purpose | Populated at |
|---|---|---|
| §3 — Framework document summary | What is being accepted (the methodology's scope and limitations) | Drafting |
| §4 — Approval chain | Who has authority, and what each role's position is | Drafting |
| §5 — Decision and acceptance scope | The Methodology Owner's decision and the scope of acceptance | Signature |
| §6 — Signature block | Who signed, when, and with what authority | Signature |
| §7 — Document control | Version, status, supersedes, next review | Drafting + signature |
| §8 — Open items | What is pending, what is deferred | Drafting |

**The record is short by design.** The methodology is documented in the pipeline; the record is the formal acceptance. **The record points to the framework document; the framework document provides the observations.**

---

## 3. Framework document summary

**The framework document's status recommendation is ACCEPTED as a demonstration of IFRS 9 methodology.** This is the **MAP Validator's recommendation** to the Methodology Owner.

**The scope:** PD/LGD/EAD construction, IFRS 9 staging, per-stage and probability-weighted ECL, register guard behavior.

**The key limitations** (from FM-CFR-001 §3.3, L1–L10):

- **L1:** PD-LGD correlation is constructed (not empirical).
- **L2:** SICR uses top-decile proxy (not true SICR).
- **L6:** PD cap masks tail convexity (quantified in NB04; limitation disclosed in MAP-STF-FM-001 §7).
- **L7:** Fairness not tested (no protected attributes in dataset; limitation disclosed in MAP-STF-FM-001 §7).
- **L8:** U.S. data only (not validated for Canada; limitation disclosed in MAP-STF-FM-001 §7).

**The framework document's assessment:** the methodology is defensible **as a demonstration** — the limitations are documented, quantified, and scoped. A production implementation would need the P-LC dependencies from the Remediation Plan (MAP-RP-001).

**The framework document is the evidence base.** This record cites the framework document's assessment; it does not restate it as the record's own judgment.

**On the citation chain.** This record's §3 is a summary of the framework document's §7 (framework observations) and FM-CFR-001 §3.3 (limitations). The three documents are chained: the CFR records the limitations, the framework document interprets them, and this record summarizes the interpretation. **A reader who wants the full picture reads all three** — the summary in this record is not a substitute for the source documents.

---

## 4. Approval chain and decision authority

### 4.1 — The approval chain

IFRS 9 Modeler (NB01–NB03) → MAP Validator (Framework document) → Internal Audit (NB04) → Methodology Owner (final sign-off)


### 4.2 — Positions and authority

The table below records each role's **position** (what they recommend, attest, or acknowledge) and **authority** (what they are authorized to do). The "Decision" column is filled only by the Methodology Owner, after signature.

| Role | Name | Authority | Position | Decision |
|---|---|---|---|---|
| IFRS 9 Modeler | `[ANALYST: name]` | Constructs parameters; cannot self-approve | **Attests** — the methodology is as described in NB01–NB04 | — |
| MAP Validator | `[ANALYST: name]` | Authors framework document; cannot sign off | **Recommends** — ACCEPT as a methodology demonstration (MAP-STF-FM-001 §7) | — |
| Internal Audit Lead | `[ANALYST: name]` | Documents compliance; cannot override | **Attests** — the compliance report is complete as documented | — |
| Methodology Owner | `[ANALYST: name]` | Final sign-off | **Decides** — see §5 | `[ANALYST: decision — ACCEPT / CONDITIONAL / REJECT]` |

**On the "Position" vs "Decision" distinction.** The Modeler, Validator, and Audit Lead do not make decisions about the methodology's acceptance — they make *positions* (attestations and recommendations) that the Methodology Owner considers. **Only the Methodology Owner has *decision* authority.** The v1.3 record's "Decision" column conflated the two: it recorded `ACCEPT` for the MAP Validator, which is the Validator's recommendation, not a decision by the Validator. The v1.4 table separates them.

**On the difference from Lending Club's decision authority matrix.** Lending Club's matrix has the MRC Chair as the sole decision-maker; the other roles make positions. Fannie Mae's matrix has the Methodology Owner as the sole decision-maker; the other roles make positions. **The structures are parallel — different titles, same governance discipline.** A reader who compares the two matrices should see the parallel, not a substantive difference.

### 4.3 — The three options available to the Methodology Owner

| Option | Meaning | Consequences |
|---|---|---|
| **ACCEPT** | Accept the methodology as a demonstration | The methodology may be cited in future work as an accepted IFRS 9 methodology demonstration, subject to the limitations disclosed in MAP-STF-FM-001. |
| **CONDITIONAL** | Accept with conditions (e.g., "accept, but require L2 and L7 to be documented more prominently") | The methodology is accepted subject to the stated conditions. The conditions must be specified in §5. |
| **REJECT** | Reject the methodology (e.g., "the constructed parameters are not defensible as a demonstration") | The methodology is not accepted. The record notes the rejection; the analyst may revise the methodology and resubmit. |

**A note on the v1.3 record's "likely outcome" and "unlikely" language.** The v1.3 record said CONDITIONAL was "the likely outcome" and REJECT was "unlikely." **The v1.4 record does not include that language.** The Methodology Owner's authority is to decide; the record's role is to present the options and their consequences, not to predict the decision. If the analyst wants to note a likely outcome, it belongs in a covering memo, not in the Approval Record.

**On the "REJECT" option's meaning.** A REJECT is not a rebuke of the analyst — it is a determination that the methodology, as presented, is not acceptable as a demonstration. **A REJECT does not block future work** — the analyst can revise the methodology (e.g., by acquiring current credit characteristics for L2, or by recalibrating the PD cap for L6) and resubmit. The record's §8 (open items) is where the resubmission process would be noted.

---

## 5. Decision and acceptance scope

**Record state:** This section is empty until the Methodology Owner's decision is recorded and signed. The fields below are populated by the Methodology Owner at signature time.

**The Methodology Owner's decision:** `[ANALYST: decision — ACCEPT / CONDITIONAL / REJECT]`

**The rationale:** `[ANALYST: rationale — required for all decisions; critical for REJECT]`

**The conditions (if CONDITIONAL):** `[ANALYST: conditions — specified in full; each condition must have an owner and a due date]`

**The acceptance scope (if ACCEPT or CONDITIONAL):** The following uses are permitted, subject to the limitations disclosed in MAP-STF-FM-001:

- **Methodology demonstration and training material** — the methodology may be used to illustrate IFRS 9 concepts.
- **Reference implementation for IFRS 9 methodology development** — the methodology may be used as a starting point for a production implementation (subject to the excluded uses below).
- **Regulatory documentation example** — the methodology may be cited as an example of documentation discipline, with the limitations disclosed.

**The following uses are excluded, regardless of the decision:**

- **Production ECL calculation** — requires the P-LC dependencies from MAP-RP-001.
- **Capital adequacy calculation** — requires production-grade parameters and RWA.
- **Canadian portfolio validation** — requires Canadian data.
- **Fairness testing** — requires protected attributes.

**The decision is the record's binding content.** **The acceptance is scoped — it is not a blanket approval.** If §5 is empty, the record is a draft; if §5 is filled and §6 is signed, the record is binding.

**On the acceptance scope's meaning.** The four excluded uses are **not** conditions on the acceptance — they are **scope exclusions** that hold regardless of whether the Methodology Owner ACCEPTs or REJECTs. **A reader who sees "ACCEPT" and concludes "the methodology is approved for production" has misread the scope** — the acceptance is for demonstration, training, and reference use only, and production use is excluded.

---

## 6. Signature block

**The signature block records who signed, when, and with what authority.** The "Position" column repeats the position from §4.2; the "Signed" column records the signature. The "Decision" column is filled only by the Methodology Owner.

| Role | Name | Position | Signature | Date | Decision |
|---|---|---|---|---|---|
| IFRS 9 Modeler | `[ANALYST: name]` | Attests | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| MAP Validator | `[ANALYST: name]` | Recommends ACCEPT | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| Internal Audit Lead | `[ANALYST: name]` | Attests | `[ANALYST: signature]` | `[ANALYST: date]` | — |
| Methodology Owner | `[ANALYST: name]` | Decides | `[ANALYST: signature]` | `[ANALYST: date]` | `[ANALYST: decision]` |

**The signature block is the record's formal completion.** Without signatures, the methodology is a demonstration without endorsement. The signatures convert the demonstration into an accepted methodology.

---

## 7. Document control

| Field | Value |
|---|---|
| Version | 1.4 |
| Status | Draft pending Methodology Owner sign-off |
| Supersedes | v1.3 (structural revision; no change to the approval's meaning) |
| Next review | At next parameter version bump (if a parameter changes, a full re-run and a new Approval Record are required) |
| Related documents | MAP-STF-FM-001 (framework document), FM-CFR-001 (Consolidated Findings Register), MAP-RP-001 (Remediation Plan) |
| Record ID | `[ANALYST: record_id]` |

**On the next review.** Unlike the Lending Club record, the Fannie Mae record does not have a calendar schedule for review. The methodology is stable, so the record is stable. A review is triggered only by a parameter change (e.g., if the PD cap is removed, a full re-run and a new Approval Record are required).

**On the v1.3 → v1.4 supersession.** The v1.4 record supersedes v1.3. **The supersession is structural** — the v1.4 separates the "Position" column from the "Decision" column, removes the "likely outcome" language, and clarifies the acceptance scope. **The approval's meaning is unchanged** — both v1.3 and v1.4 record the Methodology Owner's decision on whether to accept the methodology as a demonstration. A reader comparing the two versions should see the structural difference, not a substantive change in the record's purpose.

---

## 8. Open items and dependencies

### 8.1 — Open items

| # | Item | Owner | Due | Routing |
|---|---|---|---|---|
| 1 | **Methodology Owner sign-off** | Methodology Owner | `[ANALYST: date]` | Methodology Owner review |
| 2 | **Production-path decision** | Out of scope for the demonstration | — | The P-LC dependencies are documented in MAP-RP-001; a production team would need to commit to them |
| 3 | **Next parameter version bump** | Methodology Owner | If a parameter changes | A full re-run and a new Approval Record |

**On the "production-path decision" being out of scope.** The v1.3 record listed the production-path decision as an open item. The v1.4 record clarifies: **it is out of scope for this record.** The record is about methodology acceptance; the production path is a separate decision that would be made by a production team, with its own governance. **A reader who sees "production-path decision" in the open items list and concludes it is this record's open item has misread the record's scope.**

**On the "next parameter version bump" being an open item.** The v1.3 record listed it as an open item. The v1.4 record keeps it, because it is a **contingent open item** — it does not require action now, but it will require action if a parameter changes. **A reader who sees it and wonders "when will this happen?" should understand that the answer is "when a parameter changes," not "on a specific date."**

### 8.2 — Dependencies

| Category | Dependency |
|---|---|
| Upstream | MAP-STF-FM-001 (source of the framework observations), FM-CFR-001 (source of the limitations), MAP-RP-001 (source of the P-LC dependencies) |
| Downstream | Production implementation (if pursued), methodology citation in future work |
| Reproducibility | The record's version is recorded in the Methodology Owner's decision log |
| Cross-project | The Lending Club Approval Record (LC-AR-001) is a **deployment authorization**; this record is a **methodology acceptance**. The two records are parallel in structure but different in meaning — see §1 |

---

*End of document.*
