# Approval Record — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-AR-001  
**Version / Status:** 0.1 / Template  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Approval Record (AR)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

> **This is a template.** The approval record is the document where the Methodology Owner records the approval decision. It is **not draftable from the notebook output** — the approval is a human action, and the notebooks are forbidden from producing it (claim-authorization discipline). This template names the fields; the analyst supplies the content. **Every field marked `[ANALYST: ...]` is a human action, not a notebook computation.**

---

## 1. Purpose and scope

This Approval Record documents the Methodology Owner's decision on the Fannie Mae IFRS 9 ECL framework: whether the framework is accepted as a methodology demonstration, what conditions (if any) attach to the acceptance, and what the review cadence is.

**This is not a Validation Opinion.** The framework has not been independently validated — it is a methodology demonstration. The approval record documents the Methodology Owner's acceptance of the demonstration; it does not certify the framework for production.

---

## 2. Approval scope

| Field | Value |
|---|---|
| **What is being approved** | The Fannie Mae IFRS 9 ECL framework as a methodology demonstration |
| **What the approval does NOT cover** | Production deployment, IFRS 9 provisioning, Canadian portfolios, or any use of the framework's outputs as a forecast |
| **The approval basis** | The evidence in FM-DS-001, FM-FDL-001, FM-MD-001, FM-DCR-001, FM-CFR-001, MAP-STF-FM-001, and the FM pipeline's artifacts |
| **The approval authority** | Methodology Owner (FM-AR-001) |

**The approval is scoped.** It covers the framework *as a demonstration*. It does not cover any use of the framework's outputs. **A reader who sees "approved" and concludes "the framework is production-ready" has misread the scope** — the scope is in this section, and it says what the approval does and does not cover.

`[ANALYST: insert the approval scope statement in the analyst's voice. The table above is the factual scope; the analyst's framing is the paragraph that says what a reader should take away. If the framing is "approved as a demonstration, not as a model," say that; if it is "approved with conditions," state the conditions here or reference §5.]`

---

## 3. Approval chain

The approval chain for the framework:

| Role | Responsibility | Approval required? |
|---|---|---|
| IFRS 9 Modeler | Builds the pipeline; proposes the framework | No (proposer) |
| Validator | Reviews the framework; assesses the evidence | Advisory |
| Methodology Owner | Approves or rejects the framework | **Yes** |

**The Methodology Owner is the sole approval authority.** The validator's role is advisory — a departure from the Lending Club model, where the MRC Chair approves on the validator's recommendation. The difference is documented in FM-DCR-001 §8.

`[ANALYST: insert the names and roles of the approvers. The chain above is the roles; the names are the analyst's contribution. If a role is vacant, state that — a vacancy is a governance gap, not an omission to be papered over.]`

---

## 4. Approval decision

| Field | Value |
|---|---|
| **Decision** | `[ANALYST: insert — Approved / Approved with conditions / Rejected / Deferred]` |
| **Rationale** | `[ANALYST: insert — the Methodology Owner's rationale for the decision]` |
| **Effective date** | `[ANALYST: insert]` |
| **Review date** | `[ANALYST: insert — the next scheduled review date; typically 12 months from the effective date, or at the next MAJOR version bump, whichever comes first]` |

**On the rationale.** The rationale is the Methodology Owner's reasoning, not a summary of the evidence. The evidence is in FM-CFR-001 and the framework document; the rationale is *why* the Methodology Owner reads the evidence as supporting the decision. **A rationale that says "the evidence supports approval" is not a rationale** — it is a restatement of the decision. A rationale that says "the evidence supports approval because the framework's limitations are all documented, quantified, and within the framework's stated scope" is a rationale.

`[ANALYST: insert the rationale in the analyst's voice. The template provides the fields; the rationale is the analyst's judgment. If the analyst's rationale is "the framework is approved as a demonstration because its limitations are documented," state that. If the rationale has conditions attached, reference §5.]`

---

## 5. Conditions of approval

**If the decision is "Approved with conditions,"** the conditions are listed here. **If the decision is "Approved,"** this section states "no conditions."

| # | Condition | Verification | Owner | Timeline |
|---|---|---|---|---|
| C-1 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| C-2 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| C-3 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

**A condition without a verification method is a wish.** A condition without an owner is an orphan. A condition without a timeline is a deferral. **All three fields are required for a condition to be actionable.**

**If the framework has no conditions, write "No conditions."** An empty table with a section header is ambiguous — a reader cannot tell whether the conditions are none or missing.

`[ANALYST: insert the conditions, or "no conditions." If the analyst's decision is "approved with conditions," the conditions should reference the findings in FM-CFR-001 §7.1 and the remediation items in MAP-RP-001. If the decision is "approved," this section is the explicit statement that there are no conditions.]`

---

## 6. Review cadence

| Field | Value |
|---|---|
| **Scheduled review** | `[ANALYST: insert — typically 12 months from effective date]` |
| **Event-triggered review** | At the next MAJOR version bump of FM-DCR-001, FM-MD-001, or the framework itself |
| **Review owner** | Methodology Owner |
| **Review inputs** | The framework document's current state, the CFR's current state, the DCR's current state, any new findings since the last review |

**The review cadence is what keeps the approval current.** An approval without a review cadence is an approval that grows stale — the framework will change, the DCR will change, and a review that never happens cannot catch the change.

`[ANALYST: insert the review cadence reasoning. The template says "typically 12 months"; the analyst's framing is why 12 months (or whatever the actual cadence is). If the cadence is tied to a regulatory cycle, state that. If it is tied to a version bump, state that. If it is a policy default, state that.]`

---

## 7. Sign-off

| Role | Name | Date | Signature |
|---|---|---|---|
| Methodology Owner | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| Validator (advisory) | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

**The sign-off is the approval.** A record without a signature is a draft; a record with a signature is an approval. **The Methodology Owner's signature is the approval decision; the Validator's signature is the advisory opinion.** The two are different, and the record distinguishes them.

`[ANALYST: insert the names, dates, and signatures. If the Validator's advisory role is vacant or the Validator's signature is not available, state that — a governance record with an unfilled advisory role is a governance finding, not a document defect.]`

---

## 8. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-AR-001 |
| **Version** | 0.1 |
| **Status** | Template — every field is the analyst's contribution |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-RVP-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | Per §6 |

---

## 9. What this document contains, and what it does not

### What it contains

- The approval scope (what is being approved and what is not).
- The approval chain (the roles and the authority).
- The approval decision (fields to be filled).
- The conditions of approval (fields to be filled).
- The review cadence (fields to be filled).
- The sign-off (fields to be filled).
- The document control.

### What it does not contain (the `[ANALYST: ...]` markers)

- **Every field in §2 through §7.** The approval record is entirely the analyst's contribution — the decision, the rationale, the conditions, the cadence, the names, the dates, the signatures.
- **§8** — the document owner's name and role.

**The template is the whole document.** Unlike the previous four documents, which had notebook-sourced content, the approval record has no notebook-sourced content at all. **The notebook output supports this document's *existence*** (the DCR's §5.2 says "the Methodology Owner decides"; the framework document's §6 says "the analyst writes the Validation Opinion") — but the *content* of the approval is a human action.

**When the markers are filled, the document is an approval record.** Until then, it is a template that names the fields the approval requires.
