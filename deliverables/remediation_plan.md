# Model Risk Remediation Plan — Package Level

**Document ID:** MAP-RP-001  
**Version / Status:** 0.1 / Draft — pending analyst completion of the marked sections  
**Scope:** Cross-project (Lending Club + Fannie Mae)  
**Findings in scope:** 10 (5 LC, 5 FM) — selected from the two project-level registers  
**Roadmap horizon:** 6 months from approval  
**Owner:** `[ANALYST: name, role]`  
**Companion documents:** [`01_remediation_plan.md`](01_remediation_plan.md) (if present) · [`02_validation_report_lc.md`](02_validation_report_lc.md) · [`03_stress_testing_framework_fm.md`](03_stress_testing_framework_fm.md) · [`04_ai_interaction_record.md`](04_ai_interaction_record.md) · [`05_reproducibility_record.md`](05_reproducibility_record.md) · [`EXECUTIVE_SUMMARY.md`](EXECUTIVE_SUMMARY.md)

**Project-level plans:**
- LC: [`../lending-club-model-validation/governance/remediation-plan.md`](https://github.com/MiltonGreat/credit-risk-model-validation/blob/main/governance/remediation-plan.md) (the LC project-level plan; 8 actions, R-LC-01 through R-LC-08)
- FM: [`../fannie-mae-ifrs9-framework/governance/remediation-plan.md`](https://github.com/MiltonGreat/fannie-mae-ifrs9-portfolio/blob/main/governance/remediation-plan.md) (the FM project-level plan; 13 findings, FM-R-01 through FM-R-13)

---

## Findings at a glance

The package-level plan carries **10 findings** — 5 from the Lending Club project, 5 from the Fannie Mae project. The findings are the ones the two projects' registers identify as most material to their verdicts. **The package-level plan does not replace the project-level plans; it is a cross-project view of the findings both projects track.**

| # | Finding ID | Source project | Source register ID | Severity | Status | Project-level action |
|---|---|---|---|---|---|---|
| 1 | **LC-R-01** | Lending Club | F-03.03 | CRITICAL | OPEN | R-LC-01 |
| 2 | **LC-R-02** | Lending Club | `[ANALYST: confirm — F-03.02 (split)?]` | HIGH | OPEN | `[ANALYST: confirm — R-LC-02?]` |
| 3 | **LC-R-03** | Lending Club | `[ANALYST: confirm — F-03.04 (calibration)?]` | HIGH | OPEN | `[ANALYST: confirm — R-LC-03?]` |
| 4 | **LC-R-04** | Lending Club | `[ANALYST: confirm — F-03.06 (KS)?]` | HIGH | OPEN | `[ANALYST: confirm — R-LC-04?]` |
| 5 | **LC-R-05** | Lending Club | `[ANALYST: confirm — F-03.10 (IFRS 9)?]` | HIGH | OPEN | `[ANALYST: confirm — R-LC-08?]` |
| 6 | **FM-R-06** | Fannie Mae | (FM register — scenario calibration) | HIGH | OPEN | FM-R-06 |
| 7 | **FM-R-07** | Fannie Mae | (FM register — SICR-by-construction) | HIGH | OPEN | FM-R-07 |
| 8 | **FM-R-08** | Fannie Mae | (FM register — EAD ratio) | HIGH | OPEN | FM-R-08 |
| 9 | **FM-R-09** | Fannie Mae | (FM register — fairness incomplete) | HIGH | OPEN | FM-R-09 |
| 10 | **FM-R-10** | Fannie Mae | (FM register — PD cap disclosure) | MEDIUM | OPEN | FM-R-10 |

> **What this table is not.** It is not a status tracker — the live tracker is the MRM dashboard described in §7. It is a reading aid so a reviewer can see the scope of the plan in one view before reading the root-cause detail.

**On the LC-R-01 definition.** You have confirmed **LC-R-01 = F-03.03** (severe disparate impact, CRITICAL, OPEN) — the primary driver of the REJECT verdict. The other four LC findings (LC-R-02 through LC-R-05) are marked `[ANALYST: confirm]` because the LC register has more open findings than the plan selects, and the selection is the analyst's judgment. **The natural selection is the five most consequential open findings:** F-03.02 (split), F-03.04 (calibration), F-03.06 (KS), F-03.10 (IFRS 9 scope). **If the analyst wants a different five — e.g., including F-03.01 (leakage, RESOLVED) to show the full lifecycle — the IDs and the cross-reference table adjust.**

**On the FM-R-xx definitions.** The five FM findings are FM-R-06 through FM-R-10, matching the FM project-level plan's cross-project findings. The FM register has 13 findings total; the package-level plan carries five. **The five are the ones the FM project-level plan's §8 names as its cross-project selection.**

---

## 1. Purpose and scope

This plan tracks the closure of model risk findings raised across the Lending Club and Fannie Mae pipelines. **It is not a Validation Opinion** and does not itself authorize or reject any model. It records what is broken, who owns the fix, how the fix will be evidenced, and when the fix must be closed — the audit trail between a finding being raised and a finding being closed.

**The package-level plan's role.** The two project-level plans (LC `governance/remediation-plan.md` and FM `governance/remediation-plan.md`) track **all** the findings each project produced. The package-level plan tracks **the 10 findings that the two projects' verdicts most directly rest on** — the 5 LC findings that drive the REJECT, and the 5 FM findings that the FM framework's "accepted as a demonstration" verdict rests on. **The package-level plan is the cross-project view; the project-level plans are the complete record.**

### 1.1 — What the plan covers

**10 findings, grouped into 10 actions (5 LC + 5 FM).**

**LC (5 findings):**

| Finding ID | Source register ID | What it is | Severity | Project-level action |
|---|---|---|---|---|
| LC-R-01 | F-03.03 | Severe disparate impact (DI 0.5414) | CRITICAL | R-LC-01 |
| LC-R-02 | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-03 | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-04 | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-05 | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |

**FM (5 findings):**

| Finding ID | What it is | Severity | Project-level action |
|---|---|---|---|
| FM-R-06 | Scenario calibration (severity ratio 1.27× vs 1.5× target) | HIGH | FM-R-06 |
| FM-R-07 | SICR-by-construction (Stage 2 is top-decile proxy) | HIGH | FM-R-07 |
| FM-R-08 | EAD ratio outside configured band (86.95% vs 50–85%) | HIGH | FM-R-08 |
| FM-R-09 | Fairness testing incomplete (geographic proxies at high cardinality) | HIGH | FM-R-09 |
| FM-R-10 | PD cap disclosure (cap does not bind in this run) | MEDIUM | FM-R-10 |

### 1.2 — What the plan does not cover

- **The other 8 LC findings** (the LC project-level plan tracks 8 actions covering all 8 remediable findings). The package plan carries 5 of them; the other 3 are tracked in the LC project-level plan.
- **The other 8 FM findings** (the FM project-level plan tracks 13 findings). The package plan carries 5 of them; the other 8 are tracked in the FM project-level plan.
- **The 4 LC OPEN (docs) findings** (F-01.07, F-01.08, F-02.07): closed by documenting the decision, not by the package-level plan.
- **The 9 LC OPEN (scope) findings** (F-01.04–F-01.06, F-02.08, F-03.08–F-03.10): permanently OPEN until the scope changes.
- **The 3 FM LOW findings** (the FM register's data-preparation findings): documentation-only; not in the package-level plan.

**On the "10 of 21" framing.** The two project-level plans together track 21 findings (8 LC + 13 FM). The package-level plan carries 10 of them (5 + 5). **The 10 are the selection; the 21 are the record.** A reader who wants the full picture reads the two project-level plans; a reader who wants the cross-project view reads this document.

---

## 2. Issue identification and root cause analysis

### 2.1 — Root-cause categories

| Category | What it means | Which findings |
|---|---|---|
| **Conceptual** | The methodology's design is wrong or incomplete | `[ANALYST: confirm]` |
| **Methodology — proxy** | The methodology uses a proxy where the standard requires an observation | FM-R-07 (SICR-by-construction) |
| **Calibration** | The parameter is an assumption, not a calibration | FM-R-06 (scenario multipliers), FM-R-08 (EAD ratio) |
| **Data / scope** | The finding is a structural limitation of the data | FM-R-09 (fairness proxies) |
| **Documentation** | The finding is that something is not documented, not that it is wrong | FM-R-10 (PD cap disclosure) |
| **Fairness** | The finding is a disparate impact | LC-R-01 (fairness) |

### 2.2 — Root-cause detail per finding

**LC-R-01 — Severe disparate impact (F-03.03).** Region-level DI = 0.5414 (SEVERE). State features act as a region proxy for protected-class membership. The model's learned relationship between geography and default is a **proxy-validity problem**, not a coding defect. **The root cause is that the model uses a geographic proxy where the data has no direct protected attribute.** The remediation is either removing the state features or reweighting; the LC project-level plan's R-LC-01 documents the specific action.

**FM-R-06 — Scenario calibration.** The scenario multipliers (1.0×/1.8×/3.0×/4.5× for PD; 1.0×/1.2×/1.5×/2.0× for LGD) are analyst-specified heuristics with a precedent label ("the 2008 financial crisis"), not calibrated to the portfolio. The severity check (AP-05) reports the Severely Adverse ECL as 1.27× the 2008–2010 historical average — below the 1.5× target in the DCR. **The root cause is that the multipliers are assumptions, not derivations.** A production implementation would derive them from a macro-to-PD linkage.

**FM-R-07 — SICR-by-construction.** The Stage 2 population is the top-decile of non-Stage-3 loans by `pd_estimate`. The dataset contains origination characteristics only; it does not contain current credit characteristics. A true IFRS 9 §5.5.9 SICR assessment requires comparing credit risk at the reporting date to credit risk at initial recognition — which cannot be done with origination-only data. **The root cause is that the dataset lacks the current credit characteristics a true SICR assessment requires.**

**FM-R-08 — EAD ratio.** The EAD ratio is 86.95% against a configured 50–85% band; AP-03 fired. Two candidate causes: (a) the loan sample is young (avg 16 months), so prepayment/curtailment factors are near 1.0; (b) `ORG_UPB` is used as the exposure base instead of current UPB. **The root cause is not yet resolved** — the notebook flags the band breach but does not identify the cause. The plan's first action for this finding is diagnosis.

**FM-R-09 — Fairness testing incomplete.** Fairness testing on geographic proxies (state, MSA, ZIP) at cardinality 52/328/638 produces degenerate DI = 0.00%, flagged UNRELIABLE. The dataset has no direct protected attributes. **The root cause is a structural data limitation:** no protected attributes are present, and the 4/5ths rule is misapplied at high cardinality. **The finding is a limitation, not a defect.**

**FM-R-10 — PD-cap disclosure.** The PD cap (20%) does not bind in the current run because Tail Risk PD (4.29%) is below the cap. The cap-sensitivity analysis reports +0.0% understatement. The mechanism is documented as Limitation L6; the uncapped value would be reported alongside the capped value if the cap fired. **The root cause is that the cap is a stability measure that could be anti-conservative in runs where base PD is higher.**

**LC-R-02 through LC-R-05** — `[ANALYST: insert the root-cause detail for the four LC findings, using the LC project-level plan's action descriptions and the LC register's finding descriptions. Each root-cause detail should follow the pattern above: what is wrong, why, and (if the cause is not yet identified) what the first action is.]`

---

## 3. Model tiering and prioritization

Tiering is a function of **(a)** financial materiality, **(b)** complexity, and **(c)** usage scale. The two projects are methodology demonstrations, not production models, so the tier reflects **the tier the model/framework would carry if promoted to its stated intended use**, not its current demonstration status.

| Finding ID | Tier | Materiality rationale | Remediation SLA |
|---|---|---|---|
| LC-R-01 | Tier 1 (High) | Fairness finding drives the REJECT verdict; a model with SEVERE DI cannot be used for any consequential decision | 3 months |
| LC-R-02 | `[ANALYST: confirm tier]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-03 | `[ANALYST: confirm tier]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-04 | `[ANALYST: confirm tier]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-05 | `[ANALYST: confirm tier]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| FM-R-06 | Tier 2 (Medium) | Scenario severity is directional; the 1.27× ratio is an investigation trigger, not a failure | 2 months |
| FM-R-07 | Tier 2 (Medium) | SICR-by-construction is a scope limitation; it affects the interpretation of Stage 2 ECL, not the computation | 3 months (data acquisition) / documented limitation otherwise |
| FM-R-08 | Tier 1 (High) | EAD ratio outside band affects every ECL number downstream | 1 month (diagnosis) / 2 months (resolution) |
| FM-R-09 | Tier 2 (Medium) | Fairness testing is incomplete, not failed; the limitation is structural | 6 months (data acquisition) |
| FM-R-10 | Tier 3 (Low) | PD cap effect is 0.0% in the current run; monitor | 6 months (monitoring) |

### Prioritization order for the 6-month roadmap

1. **Month 1:** FM-R-08 diagnosis (EAD ratio — resolve `ORG_UPB` vs. current UPB); LC-R-03 (calibration — Platt scaling for XGBoost) `[ANALYST: confirm LC-R-03 = calibration]`.
2. **Months 1–2:** FM-R-08 resolution; LC-R-04 (KS improvement) `[ANALYST: confirm LC-R-04 = KS]`.
3. **Months 2–3:** LC-R-01 fairness remediation (remove state features or reweight); LC-R-02 (split method) `[ANALYST: confirm LC-R-02 = split]`; FM-R-06 scenario multiplier recalibration.
4. **Months 3–4:** LC-R-05 (IFRS 9 scope decision) `[ANALYST: confirm LC-R-05 = IFRS 9]`; FM-R-07 data acquisition for current credit characteristics (or documented permanent limitation).
5. **Months 4–6:** FM-R-09 protected-attribute data acquisition; FM-R-10 monitoring.

**On the prioritization.** The LC findings and the FM findings run in parallel — the two projects are independent, and their remediation can proceed on separate tracks. **The package-level plan's timeline is the union of the two projects' timelines**, not a single sequence.

`[ANALYST: confirm the prioritization order and the LC findings' tier assignments. The FM findings' prioritization is from the FM project-level plan; the LC findings' prioritization is from the LC project-level plan's R-LC-xx schedule. If the package-level plan should reorder them (e.g., by cross-project dependency), say so.]`

---

## 4. Remediation actions and milestones

Each row has an **action owner** (role), a **milestone date** (relative to plan approval), and **evidence that closure will require**.

### 4.1 — LC findings

| Finding ID | Action | Owner | Milestone | Closure evidence |
|---|---|---|---|---|
| LC-R-01 | Remove state features from feature set; rerun NB02/NB03; confirm region DI ≥ 0.80 or document why reweighting is preferable | Model Development + Model Risk | M3 | New NB03 run with DI ≥ 0.80 at FD-05-compliant cardinality, or documented reweighting methodology approved by Model Risk |
| LC-R-02 | `[ANALYST: confirm action from LC project-level plan's R-LC-02]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-03 | `[ANALYST: confirm action from LC project-level plan's R-LC-03]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-04 | `[ANALYST: confirm action from LC project-level plan's R-LC-04]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-05 | `[ANALYST: confirm action from LC project-level plan's R-LC-08 (IFRS 9 scope)]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |

### 4.2 — FM findings

| Finding ID | Action | Owner | Milestone | Closure evidence |
|---|---|---|---|---|
| FM-R-06 | Recalibrate scenario multipliers against portfolio-specific historical data or a macro-to-PD linkage; document the calibration source and target ratio (≥ 1.5× per DCR §AP-05) | Model Development | M3 | New NB04 run with severity ratio ≥ 1.5×, or a signed DCR amendment documenting why < 1.5× is accepted |
| FM-R-07 | Acquire current credit characteristics (or document a permanent limitation); recalibrate Stage 2 logic if data acquired | Model Development + Data Owner | M3 (or documented limitation) | New NB03 stage output with observed SICR, or a signed limitation in the framework's documentation |
| FM-R-08 | Diagnose the EAD ratio: confirm whether `ORG_UPB` or current UPB is the correct exposure base; recalibrate prepayment/curtailment factors if needed | Model Development | M1 (diagnosis) / M2 (resolution) | New NB03 run with EAD ratio in 50–85% band, or a documented rationale for the band definition |
| FM-R-09 | Acquire direct protected attributes (HMDA-equivalent) and rerun fairness at low cardinality | Model Risk + Data Owner | M6 | New NB04 fairness result at cardinality ≤ 50, or a documented permanent limitation |
| FM-R-10 | Monitor the PD cap effect across runs; recalibrate the cap if base PD rises | Model Development | M6 (ongoing) | A monitoring report showing cap effect across runs |

**Closure evidence is not a narrative claim.** A finding is closed when a **new pipeline run** demonstrates the closure — a new NB03 or NB04 output with the before/after comparison. Self-attestation by the remediation owner does not close a finding; independent re-validation (see §6) does.

---

## 5. Compensating controls

While permanent fixes are under development, the following compensating controls limit the damage each finding can cause. **These are bridge controls, not substitutes for remediation.**

| Finding ID | Compensating control | Owner | Duration |
|---|---|---|---|
| LC-R-01 | Do not use the model for any consequential decision. Scores may be used for research and methodology demonstration only. If a decision is required, use a documented manual override with Model Risk approval. | Model Risk | Until M3 |
| LC-R-02 | `[ANALYST: confirm from LC project-level plan]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-03 | `[ANALYST: confirm from LC project-level plan]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-04 | `[ANALYST: confirm from LC project-level plan]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| LC-R-05 | `[ANALYST: confirm from LC project-level plan]` | `[ANALYST: confirm]` | `[ANALYST: confirm]` |
| FM-R-06 | Label the Severely Adverse scenario as *directional only*; do not cite the 1.27× ratio as evidence of scenario adequacy | Model Development | Until M3 |
| FM-R-07 | Label Stage 2 as *SICR by construction* in every downstream report; do not present 9.9% as observed SICR | Model Development | Until M3 |
| FM-R-08 | Do not rely on EAD or ECL numbers until the EAD ratio is resolved. If ECL is cited, include the EAD ratio and the AP-03 warning. | Model Risk | Until M2 |
| FM-R-09 | Label fairness testing as *incomplete, not failed*; do not cite DI = 0.00% as evidence of unfairness or of fairness | Model Risk | Until M6 |
| FM-R-10 | Include the PD-cap effect in every ECL report; do not present tail ECL without the cap disclosure | Model Development | Ongoing |

**The pattern for each control:** a control that a reviewer can verify without running the pipeline. "Label the scenario directional" is verifiable from the report; "be careful with the scenario" is not.

---

## 6. Independent re-validation and sign-off

Each finding's closure requires **independent re-validation**, not self-attestation by the remediation owner. The re-validation must be performed by a reviewer who did not implement the fix, and the evidence must be a **new pipeline run**.

### 6.1 — Closure evidence template per finding

For each finding, the closure evidence package must contain:

1. **Finding ID and severity** (from the source register)
2. **Root cause** (from §2 of this plan)
3. **Action taken** (from §4 of this plan)
4. **New run ID(s) and artifact hashes** for the evidence
5. **Before / after comparison** (metric, direction, magnitude)
6. **Residual risk** (what remains after the fix)
7. **Independent reviewer name, role, date, signature**
8. **Closure approval** (Model Risk Committee or delegate)

### 6.2 — Sign-off table

> **Note on the placeholders.** The `[ANALYST: ...]` fields below are the analyst's contribution, not omissions. The plan produces the structure; the analyst produces the closure evidence and the signature. This is deliberate — a remediation plan that auto-fills its own sign-off is not an independent re-validation.

| Finding ID | Closure evidence run ID | Residual risk | Independent reviewer | Date | Approved |
|---|---|---|---|---|---|
| LC-R-01 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| LC-R-02 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| LC-R-03 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| LC-R-04 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| LC-R-05 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-06 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-07 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-08 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-09 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |
| FM-R-10 | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` | `[ANALYST: insert]` |

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

- **CRITICAL severity** (LC-R-01)
- **HIGH severity with SLA breach > 30 days**

**The LC-R-01 finding is the only CRITICAL finding in the package-level plan.** The FM findings are all HIGH or MEDIUM. **If the analyst selects a different five LC findings and any is CRITICAL, the Board-level visibility criteria adjust.**

---

## 8. Cross-reference to the project-level plans

The package-level plan's 10 findings are 10 of the 21 total across the two project-level plans. **The selection is the package-level plan's; the full record is the project-level plans'.**

| Package-level finding | Source project | Project-level plan reference |
|---|---|---|
| LC-R-01 | Lending Club | `governance/remediation-plan.md` — action R-LC-01 |
| LC-R-02 | Lending Club | `governance/remediation-plan.md` — action `[ANALYST: confirm]` |
| LC-R-03 | Lending Club | `governance/remediation-plan.md` — action `[ANALYST: confirm]` |
| LC-R-04 | Lending Club | `governance/remediation-plan.md` — action `[ANALYST: confirm]` |
| LC-R-05 | Lending Club | `governance/remediation-plan.md` — action `[ANALYST: confirm]` |
| FM-R-06 | Fannie Mae | `governance/remediation-plan.md` — action FM-R-06 |
| FM-R-07 | Fannie Mae | `governance/remediation-plan.md` — action FM-R-07 |
| FM-R-08 | Fannie Mae | `governance/remediation-plan.md` — action FM-R-08 |
| FM-R-09 | Fannie Mae | `governance/remediation-plan.md` — action FM-R-09 |
| FM-R-10 | Fannie Mae | `governance/remediation-plan.md` — action FM-R-10 |

**Why the package-level plan carries only 10 of 21 findings:** the package-level plan's selection criteria are "the findings most material to the two verdicts." The five LC findings are the ones the LC REJECT verdict most directly rests on; the five FM findings are the ones the FM "accepted as a demonstration" verdict most directly rests on. **The selection is a documented judgment, not a suppression** — the other 11 findings are tracked in the project-level plans.

---

## 9. Document control

| Field | Value |
|---|---|
| **Document ID** | MAP-RP-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Next review** | On completion of the marked sections, or at the next version bump of either project-level plan |
| **Related documents** | `02_validation_report_lc.md`, `03_stress_testing_framework_fm.md`, `04_ai_interaction_record.md`, `05_reproducibility_record.md`, `EXECUTIVE_SUMMARY.md` |
| **Project-level plans** | LC: `lending-club-model-validation/governance/remediation-plan.md`; FM: `fannie-mae-ifrs9-framework/governance/remediation-plan.md` |

---

## 10. What this document contains, and what it does not

### What it contains (from the two project-level plans and the two project READMEs)

- The 10-finding scope (5 LC + 5 FM) with the cross-reference to the source registers.
- The FM findings' root-cause analysis, tiering, actions, and compensating controls (sourced from the FM project-level plan).
- The LC-R-01 finding's root-cause analysis, tiering, action, and compensating control (sourced from the LC project-level plan's R-LC-01).
- The cross-reference to the two project-level plans.
- The sign-off template and the executive tracking fields.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§Findings at a glance** — LC-R-02 through LC-R-05's source register IDs, severities, and project-level actions. You confirmed LC-R-01 = F-03.03; the other four are `[ANALYST: confirm]`.
- **§2.2** — root-cause detail for LC-R-02 through LC-R-05.
- **§3** — tier assignments for LC-R-02 through LC-R-05.
- **§4.1** — actions, owners, milestones, and closure evidence for LC-R-02 through LC-R-05.
- **§5** — compensating controls for LC-R-02 through LC-R-05.
- **§6.2** — the sign-off table (all 10 rows).
- **§9** — the document owner's name and role.

**Once you confirm the four LC findings, the markers in §Findings at a glance, §2.2, §3, §4.1, §5, and §8 can be replaced with the LC project-level plan's content.** The sign-off table (§6.2) is the analyst's contribution regardless.

**When the markers are filled, the document is complete.** It will be the cross-project view of the two project-level plans — the document that names the 10 findings both projects' verdicts most directly rest on.
