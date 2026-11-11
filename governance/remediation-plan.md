# Model Risk Remediation Plan — Fannie Mae IFRS 9 ECL Framework

**Document ID:** MAP-RP-FM-001  
**Version:** 0.1  
**Status:** Draft — pending analyst completion of the marked rows  
**Project:** Fannie Mae IFRS 9 ECL Framework  
**Findings in scope:** 13 (from `governance/consolidated-findings-register.md`; five of the 13 are also carried by the package-level plan as FM-R-06 through FM-R-10)  
**Roadmap horizon:** 6 months from approval  
**Owner:** `[ANALYST: name, role]`  
**Companion documents:** [`consolidated-findings-register.md`](consolidated-findings-register.md), [`decision-criteria-register.md`](decision-criteria-register.md), [`risk-validation-plan.md`](risk-validation-plan.md), [`../deliverables/remediation_plan.md`](../deliverables/remediation_plan.md) (package-level plan)

---

## Findings at a glance

| # | Finding ID | Severity | Tier | SLA | Status |
|---|---|---|---|---|---|
| 1 | FM-R-01 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 2 | FM-R-02 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 3 | FM-R-03 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 4 | FM-R-04 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 5 | FM-R-05 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 6 | FM-R-06 | HIGH | `[ANALYST: insert]` | `[ANALYST: insert]` | OPEN |
| 7 | FM-R-07 | HIGH | `[ANALYST: insert]` | `[ANALYST: insert]` | OPEN |
| 8 | FM-R-08 | HIGH | `[ANALYST: insert]` | `[ANALYST: insert]` | OPEN |
| 9 | FM-R-09 | HIGH | `[ANALYST: insert]` | `[ANALYST: insert]` | OPEN |
| 10 | FM-R-10 | MEDIUM | `[ANALYST: insert]` | `[ANALYST: insert]` | OPEN |
| 11 | FM-R-11 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 12 | FM-R-12 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| 13 | FM-R-13 | `[ANALYST: insert from register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

**FM-R-06 through FM-R-10 are the five findings the package-level plan carries.** They are the ones with severity and status I can verify from the FM README's narrative: FM-R-06 is the scenario-calibration finding (severity ratio 1.27× vs. 1.5× target; AP-05 fired); FM-R-07 is the SICR-by-construction finding (Stage 2 top-decile proxy); FM-R-08 is the EAD ratio finding (86.95% vs. 50–85% band; AP-03 fired); FM-R-09 is the fairness-testing finding (geographic proxies at high cardinality; UNRELIABLE); FM-R-10 is the PD-cap disclosure finding (the cap does not bind in this run, but the mechanism is documented).

**The other eight rows (FM-R-01 through FM-R-05, FM-R-11 through FM-R-13) are the FM register's findings that the package-level plan did not carry.** Their exact content is in `governance/consolidated-findings-register.md`; I have not seen the register, so I have marked each row's content with `[ANALYST: insert from register]`. Once you share the register, every marker can be replaced.

> **What this table is not.** It is not a status tracker — the live tracker is the MRM dashboard described in §7. This table is a reading aid so a reviewer can see the scope of the plan in one view before reading the root-cause detail.

---

## 1. Purpose and scope

This plan tracks the closure of model risk findings raised by the Fannie Mae IFRS 9 ECL Framework pipeline. It is **not a Validation Opinion** and does not itself authorize or reject the framework. It records what is broken, who owns the fix, how the fix will be evidenced, and when the fix must be closed — the audit trail between a finding being raised and a finding being closed.

**The Fannie Mae project is a framework, not a validated model.** It does not have a Validation Opinion and will not have one (see [`../README.md`](../README.md) §"What this framework does NOT demonstrate"). The remediation plan is the framework's governance artifact — it tracks the framework's findings the way the LC plan tracks the model's findings, but the two plans are answering different questions:

- The **LC plan** answers: *how do we get the model from REJECT to a state where it can be re-validated?*
- The **FM plan** answers: *what does the framework need to do before it can be presented as a production-ready methodology rather than a demonstration?*

**The scope of this plan:** the 13 findings in the FM `consolidated-findings-register.md`, with FM-R-06 through FM-R-10 also carried by the package-level plan for cross-project visibility.

### Scope exclusions

- **Legal conclusions** about OSFI E-23, SR 11-7, IFRS 9, or ECOA compliance are out of scope. The plan tracks the *model risk* findings, not the regulatory determinations.
- **The Fannie Mae Validation Opinion** is out of scope — the framework is not a validated model, and the Validation Opinion is intentionally absent.
- **Canadian portfolio validation** is out of scope for this plan — it is a downstream activity, not part of closing the framework's findings.

---

## 2. Issue identification and root cause analysis

### 2.1 Root-cause categories

Each finding is categorized into one of five root-cause categories, matching the categories the LC plan uses:

| Category | What it means | Which findings |
|---|---|---|
| **Conceptual** | The methodology's design is wrong or incomplete, not its implementation | `[ANALYST: insert]` |
| **Methodology — proxy** | The methodology uses a proxy where the standard requires an observation | FM-R-07 (SICR-by-construction) |
| **Calibration** | The parameter is an assumption, not a calibration | FM-R-06 (scenario multipliers), FM-R-08 (EAD ratio) |
| **Data / scope** | The finding is a structural limitation of the data | FM-R-09 (fairness proxies), FM-R-10 (PD-cap disclosure) |
| **Documentation** | The finding is that something is not documented, not that it is wrong | `[ANALYST: insert]` |

### 2.2 Root-cause detail per finding

**FM-R-06 — Scenario calibration.** The scenario multipliers (1.0×/1.8×/3.0×/4.5× for PD; 1.0×/1.2×/1.5×/2.0× for LGD) are analyst-specified heuristics with a precedent label ("the 2008 financial crisis"), not calibrated to the portfolio. The severity check (AP-05) reports the Severely Adverse ECL as 1.27× the 2008–2010 historical average — below the 1.5× target in the DCR. **The finding is that the assumed multipliers produce a scenario the register considers insufficiently severe; the finding is not that the portfolio is safe.** The root cause is that the multipliers are assumptions, not derivations. A production implementation would derive them from a macro-to-PD linkage.

**FM-R-07 — SICR-by-construction.** The Stage 2 population is the top-decile of non-Stage-3 loans by `pd_estimate`. The dataset contains origination characteristics only; it does not contain current credit characteristics. A true IFRS 9 §5.5.9 SICR assessment requires comparing credit risk at the reporting date to credit risk at initial recognition — which cannot be done with origination-only data. The root cause is that the dataset lacks the current credit characteristics a true SICR assessment requires. The Stage 2 fraction (9.9%) is a construction, not an observed measurement.

**FM-R-08 — EAD ratio.** The EAD ratio is 86.95% against a configured 50–85% band; AP-03 fired. Two candidate causes: (a) the loan sample is young (avg 16 months), so prepayment/curtailment factors are near 1.0; (b) `ORG_UPB` is used as the exposure base instead of current UPB. The root cause is **not yet resolved** — the notebook flags the band breach but does not identify the cause. The plan's first action for this finding is diagnosis.

**FM-R-09 — Fairness testing incomplete.** Fairness testing on geographic proxies (state, MSA, ZIP) at cardinality 52/328/638 produces degenerate DI = 0.00%, flagged UNRELIABLE. The dataset has no direct protected attributes. The root cause is a structural data limitation: no protected attributes are present, and the 4/5ths rule is misapplied at high cardinality. The finding is a limitation, not a defect.

**FM-R-10 — PD-cap disclosure.** The PD cap (20%) does not bind in the current run because Tail Risk PD (4.29%) is below the cap. The cap-sensitivity analysis reports +0.0% understatement. The mechanism is documented as Limitation L6; the uncapped value would be reported alongside the capped value if the cap fired. The root cause is that the cap is a stability measure that could be anti-conservative in runs where base PD is higher.

`[ANALYST: insert root-cause detail for FM-R-01 through FM-R-05, and for FM-R-11 through FM-R-13, using the FM consolidated-findings-register. Each root-cause detail should follow the pattern above: what is wrong, why, and (if the cause is not yet identified) what the first action is.]`

---

## 3. Model tiering and prioritization

Tiering is a function of **(a)** financial materiality, **(b)** complexity, and **(c)** usage scale. The Fannie Mae framework is a methodology demonstration, not a production model, so the tier reflects **the tier the framework would carry if promoted to its stated intended use**, not its current demonstration status.

| Finding ID | Tier | Materiality rationale | Remediation SLA |
|---|---|---|---|
| FM-R-06 | Tier 2 (Medium) | Scenario severity is directional; the 1.27× ratio is an investigation trigger (IT-05), not a failure | `[ANALYST: insert SLA]` |
| FM-R-07 | Tier 2 (Medium) | SICR-by-construction is a scope limitation; it affects the interpretation of Stage 2 ECL, not the computation | `[ANALYST: insert SLA]` |
| FM-R-08 | Tier 1 (High) | EAD ratio outside band affects every ECL number downstream | `[ANALYST: insert SLA]` |
| FM-R-09 | Tier 2 (Medium) | Fairness testing is incomplete, not failed; the limitation is structural | `[ANALYST: insert SLA]` |
| FM-R-10 | Tier 3 (Low) | PD cap effect is 0.0% in the current run; monitor | `[ANALYST: insert SLA]` |
| FM-R-01 through FM-R-05 | `[ANALYST: insert from register]` | `[ANALYST: insert from register]` | `[ANALYST: insert]` |
| FM-R-11 through FM-R-13 | `[ANALYST: insert from register]` | `[ANALYST: insert from register]` | `[ANALYST: insert]` |

### Prioritization order for the 6-month roadmap

1. **Month 1:** FM-R-08 diagnosis — resolve whether `ORG_UPB` or current UPB is the correct exposure base; confirm prepayment/curtailment factor calibration.
2. **Months 1–2:** FM-R-08 resolution; `[ANALYST: insert whether FM-R-01 through FM-R-05 include a Tier 1 item that should be prioritized]`.
3. **Months 2–3:** FM-R-06 scenario multiplier recalibration (derive from macro-to-PD linkage or document a permanent limitation); FM-R-07 data acquisition for current credit characteristics (or documented permanent limitation).
4. **Months 3–4:** `[ANALYST: insert]`.
5. **Months 4–6:** FM-R-09 protected-attribute data acquisition; FM-R-10 monitoring; `[ANALYST: insert]`.

---

## 4. Remediation actions and milestones

Each row has an **action owner** (role), a **milestone date** (relative to plan approval), and **evidence that closure will require**.

| Finding ID | Action | Owner | Milestone | Closure evidence |
|---|---|---|---|---|
| FM-R-06 | Recalibrate scenario multipliers against portfolio-specific historical data or a macro-to-PD linkage; document the calibration source and target ratio (≥ 1.5× per DCR §AP-05) | Model Development | M3 | New NB04 run with severity ratio ≥ 1.5×, or a signed DCR amendment documenting why < 1.5× is accepted |
| FM-R-07 | Acquire current credit characteristics (or document a permanent limitation); recalibrate Stage 2 logic if data acquired | Model Development + Data Owner | M3 (or documented limitation) | New NB03 stage output with observed SICR, or a signed limitation in the framework's documentation |
| FM-R-08 | Diagnose the EAD ratio: confirm whether `ORG_UPB` or current UPB is the correct exposure base; recalibrate prepayment/curtailment factors if needed | Model Development | M1 (diagnosis) / M2 (resolution) | New NB03 run with EAD ratio in 50–85% band, or a documented rationale for the band definition |
| FM-R-09 | Acquire direct protected attributes (HMDA-equivalent) and rerun fairness at low cardinality | Model Risk + Data Owner | M6 | New NB04 fairness result at cardinality ≤ 50, or a documented permanent limitation |
| FM-R-10 | Monitor the PD cap effect across runs; recalibrate the cap if base PD rises | Model Development | M6 (ongoing) | A monitoring report showing cap effect across runs |
| FM-R-01 through FM-R-05, FM-R-11 through FM-R-13 | `[ANALYST: insert action per finding from the FM consolidated-findings-register]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

**Closure evidence is not a narrative claim.** A finding is closed when a **new pipeline run** demonstrates the closure — a new NB03 or NB04 output with the before/after comparison. Self-attestation by the remediation owner does not close a finding; independent re-validation (see §6) does.

---

## 5. Compensating controls

While permanent fixes are under development, the following compensating controls limit the damage each finding can cause. **These are bridge controls, not substitutes for remediation.**

| Finding ID | Compensating control | Owner | Duration |
|---|---|---|---|
| FM-R-06 | Label the Severely Adverse scenario as *directional only*; do not cite the 1.27× ratio as evidence of scenario adequacy | Model Development | Until M3 |
| FM-R-07 | Label Stage 2 as *SICR by construction* in every downstream report; do not present 9.9% as observed SICR | Model Development | Until M3 |
| FM-R-08 | Do not rely on EAD or ECL numbers until the EAD ratio is resolved. If ECL is cited, include the EAD ratio and the AP-03 warning. | Model Risk | Until M2 |
| FM-R-09 | Label fairness testing as *incomplete, not failed*; do not cite DI = 0.00% as evidence of unfairness or of fairness | Model Risk | Until M6 |
| FM-R-10 | Include the PD-cap effect in every ECL report; do not present tail ECL without the cap disclosure | Model Development | Ongoing |
| FM-R-01 through FM-R-05, FM-R-11 through FM-R-13 | `[ANALYST: insert compensating control per finding from the FM consolidated-findings-register]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

**The pattern for each control:** a control that a reviewer can verify without running the pipeline. "Label the scenario directional" is verifiable from the report; "be careful with the scenario" is not.

---

## 6. Independent re-validation and sign-off

Each finding's closure requires **independent re-validation**, not self-attestation by the remediation owner. The re-validation must be performed by a reviewer who did not implement the fix, and the evidence must be a **new pipeline run**.

### 6.1 Closure evidence template per finding

For each finding, the closure evidence package must contain:

1. **Finding ID and severity** (from `governance/consolidated-findings-register.md`)
2. **Root cause** (from §2 of this plan)
3. **Action taken** (from §4 of this plan)
4. **New run ID(s) and artifact hashes** for the evidence
5. **Before / after comparison** (metric, direction, magnitude)
6. **Residual risk** (what remains after the fix)
7. **Independent reviewer name, role, date, signature**
8. **Closure approval** (Model Risk Committee or delegate)

### 6.2 Sign-off table

> **Note on the placeholders.** The `[ANALYST: ...]` fields below are the analyst's contribution, not omissions. The plan produces the structure; the analyst produces the closure evidence and the signature. This is deliberate — a remediation plan that auto-fills its own sign-off is not an independent re-validation.

| Finding ID | Closure evidence run ID | Residual risk | Independent reviewer | Date | Approved |
|---|---|---|---|---|---|
| FM-R-01 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-02 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-03 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-04 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-05 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-06 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-07 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-08 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-09 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-10 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-11 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-12 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-13 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

---

## 7. Executive tracking and reporting

The plan is tracked in the **MRM dashboard** with the following fields per finding:

- **Status:** OPEN / IN PROGRESS / CLOSED / DEFERRED
- **Days open** (from finding raise to closure)
- **Days remaining** to SLA
- **SLA breach flag** (red / amber / green)
- **Escalation flag** (CRITICAL findings escalate to MRC immediately; HIGH findings escalate on SLA breach)
- **Owner**
- **Evidence artifact** (link to the new run's manifest)

### Monthly reporting to the Model Risk Committee

- Findings closed this month
- Findings slipping past SLA
- Findings with residual risk above threshold
- Any finding where the remediation action has changed scope

### Board-level visibility

Reserved for findings that meet the following criteria:

- **CRITICAL severity** (`[ANALYST: insert whether any FM finding is CRITICAL — I have not seen the FM register's severity distribution]`)
- **HIGH severity with SLA breach > 30 days**

**The FM framework's findings are all HIGH or MEDIUM in the rows I can verify.** If the FM register contains a CRITICAL finding, it should be added to the Board-level visibility criteria above.

---

## 8. Cross-reference to the package-level plan

The package-level [`../deliverables/remediation_plan.md`](../deliverables/remediation_plan.md) (MAP-RP-001) carries five FM findings (FM-R-06 through FM-R-10) alongside five LC findings (LC-R-01 through LC-R-05). This project-level plan covers **all 13 FM findings**, of which five are cross-referenced to the package-level plan.

**The cross-reference table:**

| FM finding | Package-level plan ID | Package-level scope |
|---|---|---|
| FM-R-06 | FM-R-06 | Scenario calibration |
| FM-R-07 | FM-R-07 | SICR-by-construction |
| FM-R-08 | FM-R-08 | EAD ratio |
| FM-R-09 | FM-R-09 | Fairness testing incomplete |
| FM-R-10 | FM-R-10 | PD-cap disclosure |
| FM-R-01 through FM-R-05, FM-R-11 through FM-R-13 | Not carried by the package-level plan | Tracked in this project-level plan |

**Why the package-level plan carries only five FM findings:** the package-level plan's selection criteria are "closeable within a 6-month horizon with identifiable evidence." The five FM findings in scope are the ones that met the criteria. The other eight are documented in the FM consolidated-findings-register and tracked in this project-level plan, but they are not part of the cross-project roadmap. **This is a documented selection, not a suppression.**

---

## 9. Document control

| Field | Value |
|---|---|
| **Document ID** | MAP-RP-FM-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked rows |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Next review** | On completion of the marked rows, or at the next version bump of the FM consolidated-findings-register |
| **Related documents** | [`consolidated-findings-register.md`](consolidated-findings-register.md), [`decision-criteria-register.md`](decision-criteria-register.md), [`risk-validation-plan.md`](risk-validation-plan.md), [`../deliverables/remediation_plan.md`](../deliverables/remediation_plan.md) |
| **Classification** | Internal |

---

## 10. What this document contains, and what it does not

### What it contains (from the notebook output and FM README)

- The plan structure: findings-at-a-glance, root-cause analysis, tiering, actions/milestones, compensating controls, re-validation, executive tracking, cross-reference to the package-level plan.
- The five FM-R-06 through FM-R-10 findings with root cause, action, owner, milestone, closure evidence, and compensating controls.
- The cross-reference to the package-level plan.
- The document control metadata.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§Findings-at-a-glance rows 1–5 and 11–13** — the eight FM findings the package-level plan does not carry (FM-R-01 through FM-R-05, FM-R-11 through FM-R-13). Their exact IDs, severities, tiers, SLAs, and statuses are in the FM `consolidated-findings-register.md`.
- **§2.2 root-cause detail** for those eight findings — the root-cause categorization and the detail paragraph per finding.
- **§3 tier assignment** for those eight findings.
- **§4 actions** for those eight findings.
- **§5 compensating controls** for those eight findings.
- **§6.2 sign-off table** — all 13 rows, because the sign-off is the analyst's contribution by design.
- **§7 Board-level visibility criterion** — whether any FM finding is CRITICAL.

**Once you share the FM `consolidated-findings-register.md`, the markers in the first, second, third, fourth, and fifth groups can be replaced with the register's content.** The sixth group (sign-off) is the analyst's contribution regardless of the register.

**When the markers are filled, the document is complete.** It will be structurally complete now — a reviewer can read it and see the plan's shape, the reasoning for the five findings I can verify, and the placeholders where the register's other eight findings belong.
