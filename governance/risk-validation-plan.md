
**The chain is a parameter-provenance chain, not a verdict-provenance chain.** Each notebook asserts its upstream contract (`notebook_03_parameters.json` for NB04–NB06), but the verdict is authored outside the pipeline.

**On the methodological difference between the two projects.** Lending Club validates a **model** — the verdict is in the pipeline (NB03) because the model lives in the pipeline. Fannie Mae validates a **methodology** — the verdict is in the framework document because the methodology lives in the parameter set, and the framework document is where the parameter set is assessed.

**An interviewer who reads both will ask "why is the verdict in a different place?"** The answer is: because the two projects ask different questions. A **model** has to be validated by an independent notebook (NB03) to prevent the developer from approving their own work. A **methodology** can be validated by an independent author (the framework document's validator) because the parameters are specified, not learned — there is no self-approval risk in the same sense, and the framework document's claim-authorization discipline (no notebook writes a compliance verdict) is the structural safeguard.

---

## 6. Validation governance

### 6.1 — Roles and responsibilities

| Role | Name | Responsibility | Authority |
|---|---|---|---|
| IFRS 9 Modeler | `[ANALYST: name]` | NB01–NB03 ownership; parameter construction and freezing | Implements parameters; cannot self-approve methodology |
| Model Risk Validator | `[ANALYST: name]` | Framework document (MAP-STF-FM-001); independent review | Authors the methodology observations |
| Internal Audit | `[ANALYST: name]` | NB04 ownership; governance review | Documents compliance; does not override validator |
| Methodology Owner | `[ANALYST: name]` | Final sign-off on methodology | Can approve, reject, or request changes (with documented rationale) |
| Data Steward | `[ANALYST: name]` | Custodian of Fannie Mae dataset | Controls data access and versioning |

**Separation of duties.**

- The modeler cannot validate their own methodology (NB03 vs framework document separation).
- The validator cannot approve their own observations (framework document vs Methodology Owner separation).
- Internal Audit documents but does not decide.
- The Methodology Owner is the only role with sign-off authority.

### 6.2 — Decision authority matrix

| Decision | Modeler | Validator | Audit | Methodology Owner |
|---|---|---|---|---|
| Change a parameter | Yes (with version bump) | No | Notify | Review |
| Freeze parameters | Yes | No | No | Approve |
| Write the framework observations | No | Yes | No | Review |
| Override the validation observations | No | No | No | Yes (with rationale) |
| Sign off the methodology | No | No | No | Yes |
| Deploy (if applicable) | Implement | No | No | Authorize |

**The matrix mirrors Lending Club's structure but with different roles.** The IFRS 9 Modeler is the developer equivalent; the Model Risk Validator is the validator equivalent; the Methodology Owner is the MRC Chair equivalent. The separation of duties is the same, but the artifact being approved is a **methodology**, not a model.

---

## 7. Validation schedule and milestones

### 7.1 — Phase plan

| Phase | Activity | Pipeline status | Validation status |
|---|---|---|---|
| Phase 1 — Parameter construction | NB01 + NB02 (data, features, parameters) | **Executed** | — |
| Phase 2 — Parameter freezing | NB03 (`IFRS9Config`, parameters JSON) | **Executed** | **Pending framework review** |
| Phase 3 — Scenario analysis | NB04 (stress, fairness) | **Executed** | **Pending framework review** |
| Phase 4 — Compliance | NB05 (regulatory mapping) | **Executed** | **Pending framework review** |
| Phase 5 — Power BI export | NB06 (dashboard artifacts) | **Executed** | **Pending framework review** |
| Phase 6 — Framework observations | Analyst-authored framework document (MAP-STF-FM-001) | **Drafted** | **Pending signature** |
| Phase 7 — Methodology Owner sign-off | Methodology Owner decision (FM-AR-001) | — | **Pending** |
| Phase 8 — Production handoff | If applicable, hand off to production team | — | Out of scope for this demonstration |

> **On the two status columns.** "Pipeline status" answers "did the notebook run?" "Validation status" answers "has the validation activity been signed off?" The two are different: a notebook can have executed (pipeline status: Executed) without the validation being complete (validation status: Pending framework review). Conflating them — as the v1.3 document did — makes the plan look as if the validation is complete when only the pipeline has run.

### 7.2 — Key milestones

| Milestone | Target date | Owner | Status |
|---|---|---|---|
| Framework observations drafted (MAP-STF-FM-001) | `[ANALYST: date]` | Validator | Pending |
| Methodology Owner review | `[ANALYST: date]` | Methodology Owner | Pending |
| Methodology Owner sign-off (FM-AR-001) | `[ANALYST: date]` | Methodology Owner | Pending |
| Production handoff assessment | `[ANALYST: date]` | Production team | Out of scope for this demonstration |

### 7.3 — Escalation path

| Trigger | Escalation |
|---|---|
| CRITICAL parameter finding | Immediate escalation to Methodology Owner; parameters frozen pending review |
| Register guard fires (AP-02, AP-03, AP-04) | Documented; analyst investigates; not necessarily an escalation |
| Reproducibility break (content hash differs) | Escalate to Methodology Owner; re-run required |
| New limitation discovered | Document in L1–L10 register (FM-DS-001 §8); re-validate if material |
| Methodology Owner overrides validator | Documented rationale required; re-validation on the affected dimension |

**The escalation path is different from Lending Club.** In Lending Club, a CRITICAL finding triggers immediate MRC escalation because deployment is blocked. In Fannie Mae, a CRITICAL parameter finding triggers escalation to the Methodology Owner because the parameters are frozen — changing them is a MAJOR version bump and a full pipeline re-run. **The escalation is about change control, not deployment blocking.**

---

## 8. Validation-process limitations

### 8.1 — On the relationship to other limitations

The methodology's limitations (L1–L10) are in FM-DS-001 §8. The method's open items are in FM-MD-001 §8.2. The framework's observations are in MAP-STF-FM-001 §7. **This section records only the limitations of the validation *process* — things a reader of the framework document should know about the validation itself, not about the methodology or the data.**

### 8.2 — Validation-process limitations

| ID | Limitation | Impact on the validation | Mitigation |
|---|---|---|---|
| VL-1 | **Fairness is not tested** | Cannot verify disparate impact | Documented in FM-MD-001 §6.1 and MAP-STF-FM-001 §7; longer-horizon data acquisition |
| VL-2 | **No time-based stability** | PSI not assessable | Documented in FM-DS-001 §6.2; walk-forward requires time-series data |
| VL-3 | **No recovery-data LGD** | LGD is constructed, not calibrated | Documented in FM-FDL-001 §5.3; longer-horizon data acquisition |
| VL-4 | **No macro overlay** | Economic cycle not captured | Documented in FM-MD-001 §4; scenario multipliers are analyst-specified |
| VL-5 | **SICR proxy is by construction** | Stage 2 illustrative, not observed | Documented in FM-MD-001 §5.2 and MAP-STF-FM-001 §3 |
| VL-6 | **Stage 3 collapses to default** | Not true delinquency staging | C6 limitation documented in FM-DS-001 §3 |
| VL-7 | **PD cap masks tail convexity** | Tail ECL understated pre-cap | NB04 quantifies; MAP-STF-FM-001 §7 discloses |
| VL-8 | **U.S. data only** | Not validated for Canada | Documented in FM-DS-001 §8 L8 |
| VL-9 | **The validation is performed by the same analyst who built the pipeline** | The independence claim rests on the claim-authorization discipline (no notebook writes a verdict) and the decision authority matrix (§6.2), not on organizational separation | The framework document's `[ANALYST: ...]` placeholders and the Methodology Owner's sign-off are the structural safeguards |

> **On VL-9.** The Fannie Mae project is a portfolio artifact, and the same analyst built the pipeline and will write the framework document. The independence claim rests on the **technical** separation (the pipeline writes no verdict; every status field is a placeholder) and the **governance** separation (§6.2's decision authority matrix), not on organizational independence. In a production deployment, the validator would be a separate person in a separate function. The plan states this clearly — a reader who assumes organizational independence has misunderstood the artifact.

### 8.3 — Open items

| # | Item | Current treatment | Tracking |
|---|---|---|---|
| 1 | **Framework observations** | Pending. The validator must author the observations on whether the methodology is defensible as a demonstration. | Phase 6 in §7.1 |
| 2 | **Methodology Owner sign-off** | Pending. The Methodology Owner (the MRC Chair equivalent) must record the decision and rationale. | Phase 7 in §7.1; FM-AR-001 |
| 3 | **Fairness gap** | L7 documents that fairness is untested. A production implementation would need to address this, but the demonstration scope is parameterization and staging. | FM-R-09 in MAP-RP-001 (longer-horizon) |
| 4 | **PD cap** | L6 documents that the 20% cap masks tail convexity. NB04 quantifies the pre-cap understatement. A production implementation should either remove the cap (with a separate tail-risk analysis) or justify 20% explicitly. | FM-R-10 in MAP-RP-001 |
| 5 | **SICR proxy** | L2 and F-07 document that Stage 2 is top-decile construction, not true SICR. A production implementation would need current credit characteristics. | FM-R-07 in MAP-RP-001 |

**On the routing.** Each open item is routed to a Phase in §7.1 (for items that are part of the validation lifecycle) or to a MAP-RP-001 finding (for items that are remediation work). The routing makes it clear which items are pending the validator's work and which are pending the analyst's work.

---

*End of document.*
