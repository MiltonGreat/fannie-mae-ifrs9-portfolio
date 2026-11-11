# Risk & Validation Plan — Fannie Mae IFRS 9 ECL Framework

**Document ID:** FM-RVP-001  
**Version / Status:** 0.1 / Draft  
**Project:** Fannie Mae — IFRS 9 ECL methodology demonstration  
**Document type:** Risk & Validation Plan (RVP)  
**Companion documents:** FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001

---

## 1. Purpose and scope

This RVP records the **internal self-assessment plan** for the Fannie Mae IFRS 9 ECL framework: the checks the pipeline performs on itself, the criteria they apply, the responsibilities for them, and the evidence they produce.

**This is not a second-line validation plan.** The Fannie Mae framework is a **first-line methodology demonstration, not a validated model.** No independent second-line validation has been performed, and none is planned. The FM `README` says this explicitly; the DCR's §8 says the validator's role is "to assess the framework, not to approve DCR threshold changes"; the CFR's §1 says the framework's findings are "limitations, not defects." **All three are consistent with each other, and this RVP is consistent with all three.**

**What the RVP is:** the document that names the pipeline's self-checks, states their criteria, and assigns responsibility. It is what makes the pipeline's self-assessment **auditable** — a reader who wants to know "what does the pipeline check about itself, and how do I verify a check was performed?" reads this document.

**What the RVP is not:** a claim that the framework has been validated. A framework that plans its own self-assessment is a well-governed framework; a framework that claims a second-line validation it did not have is overclaiming. The RVP is the first; it is not the second.

---

## 2. The six self-assessment components

The pipeline's internal self-assessment has six components. Each is enforced in code, produces evidence in an artifact, and is documented in a companion FM document.

| # | Component | Enforced in | Evidence produced | Documented in |
|---|---|---|---|---|
| 1 | Register guards | NB03, NB04 | WARNING / ERROR in audit log; status in DCR | FM-DCR-001 §3.3 |
| 2 | Pre-flight `FileNotFoundError` checks | NB01–NB06 | `FileNotFoundError` if upstream missing; run does not proceed | FM-CAR-001 §5 |
| 3 | Reconciliation assertions | NB05, NB06 | `AssertionError` if the NB04↔NB03 run IDs mismatch | FM-CAR-001 §5 |
| 4 | Audit-log completeness | NB01–NB06 | `logs/*_audit_trail_*.log`, one per notebook per run | FM-CAR-001 §4.3 |
| 5 | Claim-authorization self-checks | NB05 | `AssertionError` if the report contains verdict tokens | FM-CAR-001 §4; FM-MD-001 §5 |
| 6 | Manifest | NB05 | `manifest_pipeline_v1_2.csv`, one row per artifact | FM-CAR-001 §4 |

**The six components are not independent of each other** — they form a chain. The pre-flight check ensures that the notebook's upstream is present; the notebook then does its work; the register guards and reconciliation assertions check the work's outputs; the audit-log completeness record makes the work reproducible; the claim-authorization self-check ensures the work does not over-claim; the manifest makes the whole chain auditable.

**A failure at any link breaks the chain.** If the pre-flight check is bypassed, the notebook runs on stale data. If the register guard is bypassed, a parameter defect is undetected. If the audit log is missing, the run is not reproducible. If the claim-authorization self-check is bypassed, the report over-claims. If the manifest is missing, the whole chain is unverifiable.

---

## 3. Component 1 — Register guards

### 3.1 — What they check

The register guards are automated threshold checks applied to the pipeline's parameters:

| Guard | Metric | Target band | Failure band | Source |
|---|---|---|---|---|
| **AP-02** | Average LGD | 40%–60% | 30%–70% | DCR §3.3 |
| **AP-03** | EAD ratio | 60%–75% | 50%–85% | DCR §3.3 |
| **AP-04** | ECL ratio of EAD | 1%–3% | 0.5%–5% | DCR §3.3 |
| **AP-05** | Severity ratio vs. 2008–2010 | ≥ 1.5× | < 1.0× | DCR §3.4 |

### 3.2 — When they fire

| Guard status | What fires | What the analyst does |
|---|---|---|
| Within target band | No log entry | Nothing |
| Outside target, inside failure band | `WARNING` in the audit log | Document the value; investigate if unexpected |
| Outside failure band | `ERROR` in the audit log | Document; escalate if unexpected |

### 3.3 — Where the results live

The measured values are in the DCR's §4 (results against criteria), not in this document. **The RVP describes *what the guards check*; the DCR describes *what they measured*.** A reader who wants to know "did AP-03 fire?" reads the DCR §4.1; a reader who wants to know "what does AP-03 check?" reads this section.

### 3.4 — Who is responsible

| Role | Responsibility |
|---|---|
| IFRS 9 Modeler | Implements the guard checks |
| Validator | Reviews the guard results (advisory, per DCR §8) |
| Methodology Owner | Accepts the guard results as part of approval (per FM-AR-001 §4) |

`[ANALYST: insert the analyst's framing of the register guard's role in self-assessment. The guards are automated; the analyst's framing is what the automation does and does not do. Automation does the arithmetic; it does not do the interpretation. The interpretation is the analyst's contribution, and the RVP should say so.]`

---

## 4. Component 2 — Pre-flight `FileNotFoundError` checks

### 4.1 — What they check

Every notebook asserts the presence of its upstream artifacts before proceeding. If any required file is missing, the notebook raises `FileNotFoundError` and does not run.

| Notebook | Required upstream artifacts |
|---|---|
| NB01 | Raw source (`Fannie Mae.zip` → `train4.csv`) |
| NB02 | Clean frame (`fannie_mae_final_clean.csv`), feature inventory (`feature_inventory.csv`) |
| NB03 | Clean frame, feature inventory |
| NB04 | Clean frame, NB03 parameters (`notebook_03_parameters.json`) |
| NB05 | Clean frame, assumptions register, NB03 parameters, NB04 results |
| NB06 | Clean frame, NB03 parameters, NB04 results |

### 4.2 — What "fail loudly" means

The pre-flight check is not a warning; it is a stop. **A missing upstream file halts the notebook.** The pipeline does not fall back to synthetic data, default values, or a previous run's artifacts. **The no-synthetic-fallback rule** (documented in the LC pipeline's `data_quality_summary.json` and adopted in the FM pipeline's NB01) is the governing principle: a missing file is a stop-the-pipeline event, not a substitute-and-proceed event.

**Why this matters.** A pipeline that silently substitutes produces artifacts that look complete but are not. A pipeline that stops loudly produces a clear failure the analyst can investigate. **The FM pipeline's preference for the loud failure is the framework's most important engineering property** — it is what makes the self-assessment auditable, because every artifact in `outputs/` was produced by a run whose upstreams all existed.

### 4.3 — Where the evidence lives

The pre-flight checks are in the notebook code (each notebook's Phase 2 or equivalent). The audit log records the successful load (`Loaded data/...: shape=... hash=...`). **The evidence for a pre-flight check is the load line in the audit log** — a run that produced an artifact has a log line showing the artifact's upstream was loaded.

`[ANALYST: insert the analyst's framing of the "fail loudly" principle. The principle is in the LC pipeline; the FM pipeline adopts it. The analyst's framing is why this matters for the FM project specifically — a framework that quietly fell back would produce a demonstration on fake data, which is worse than no demonstration. The FM README's "what this framework does NOT demonstrate" section is where the principle's consequence lives.]`

---

## 5. Component 3 — Reconciliation assertions

### 5.1 — What they check

NB05 and NB06 assert that the artifacts they read were produced by a coherent chain of runs. The assertion:

```python
assert nb04['upstream_nb03_run_id'] == nb03['run_id'], (
    f"Contamination risk: NB04 built on NB03 run {nb04['upstream_nb03_run_id']} "
    f"but NB05 is reading NB03 run {nb03['run_id']}."
)
```

If the two run IDs differ, the notebook raises `AssertionError` and does not proceed.

### 5.2 — What "contamination" means

"Contamination" in this context is not fraud; it is **a mixing of artifacts from different runs.** If NB03 ran, then NB04 ran, then NB03 re-ran (e.g., after a parameter change), then NB05 runs — NB05 would be reading NB03's new parameters and NB04's old results. **The numbers would look plausible but would not correspond to any coherent model.**

The reconciliation assertion catches this. It is the mechanism that makes "the artifact chain is coherent" a **verifiable claim** rather than an assumption.

### 5.3 — Where the evidence lives

The assertion is in the notebook code (NB05's Phase 3, NB06's Section 1). The run IDs are in each notebook's JSON artifact (`notebook_03_parameters.json` has `run_id`; `notebook_04_results.json` has both `run_id` and `upstream_nb03_run_id`). **The evidence for the reconciliation check is the two run IDs in the two JSONs** — a reader can verify them by opening the files.

`[ANALYST: insert the analyst's framing of the reconciliation assertion. The mechanism is a two-line check; the framing is why it matters — it is what makes "the run IDs are consistent" a property a reader can verify rather than trust. The FM DCR's §6 and the FM CFR's §6 both cite this assertion as the mechanism by which the chain is auditable.]`

---

## 6. Component 4 — Audit-log completeness

### 6.1 — What they record

Every notebook writes a timestamped audit log to `logs/`. The log records:

- The run ID, notebook version, and start time.
- Every artifact load (`Loaded data/...: shape=... hash=...`).
- Every decision (via the decision log or the notebook's print statements).
- Every artifact save (`Saved: ... hash=...`).
- The run's completion status.

### 6.2 — What "completeness" means

The audit log is **complete** if a reader can reconstruct what the notebook did from the log alone — without opening the notebook's code. **The FM pipeline's logs are complete in this sense**: they record the load, the decision points, the save, and the completion. A reviewer can read a log and see the run's arc.

**What the logs do not record:** the analyst's interpretation, the analyst's judgment, or the analyst's sign-off. Those are in the FM documents (DCR, CFR, AR), not the logs. **The logs record what the notebook computed; the documents record what the analyst decided.**

### 6.3 — Where the evidence lives

The logs are in `logs/`. Each notebook has its own log file, named `{stage}_audit_trail_{run_id}.log`. **A reviewer who wants to verify a run reads the log** — the log is the run's evidence.

**The FM NB01/NB02 log-prefix collision is a known issue.** Both notebooks use `eda_audit_trail_*.log` as the prefix, so a reviewer finding a log by filename alone cannot tell whether it belongs to NB01 or NB02. This is documented in FM-CAR-001 §4.3 as an open item.

`[ANALYST: insert the analyst's framing of the audit log's role. The log is evidence; the framing is what the log's completeness means for the framework's defensibility. A log that records every decision is what makes the framework's evidence chain auditable; a log that records only the start and end is not. The FM pipeline's logs are complete because they record the decisions, not just the boundaries.]`

---

## 7. Component 5 — Claim-authorization self-checks

### 7.1 — What they check

NB05 asserts that the compliance report it writes contains no verdict tokens and that every compliance status is an `[ANALYST: ...]` placeholder:

```python
assert "[ANALYST: Executive Summary" in report_text, (
    "Claim-authorization violation: executive summary must be a placeholder."
)

for token in ["Compliant", "Non-Compliant", "PASS", "FAIL",
              "Adequate", "Inadequate", "Validated"]:
    assert token not in report_text, (
        f"Claim-authorization violation: '{token}' token found in report."
    )
```

### 7.2 — What "claim-authorization" means

The notebook is authorized to **compute**; it is not authorized to **declare**. A compliance report that says "the methodology is compliant" is over-claiming; a report that says "[ANALYST: insert status]" is a scaffold for the analyst's declaration.

**The self-check is the enforcement.** Without it, a future contributor could add a "Compliant" line and the report would look authoritative. With it, the contribution fails the self-check and the report is not written.

### 7.3 — Where the evidence lives

The self-check is in NB05's Phase 7. The report itself (`outputs/reports/ifrs9_compliance_report_v1_2.txt`) contains the placeholders. **The evidence for the self-check is the report's contents** — a reader who opens the report sees the `[ANALYST: ...]` placeholders, which is the self-check's outcome.

**The self-check's success message in v1.2 contained the word "PASS"** — the very token the assertion forbids. The v1.3 revision changed the message to "completed, no forbidden tokens." **The v1.3 change is the fix, and it is documented in the FM README's claim-authorization section.**

`[ANALYST: insert the analyst's framing of the claim-authorization self-check. The mechanism is a code assertion; the framing is why the FM project needs it — the framework produces evidence, the analyst produces the verdict, and the self-check is what keeps the two separate. The FM README's claim-authorization section is the definitive statement; the RVP's framing should reference it.]`

---

## 8. Component 6 — Manifest

### 8.1 — What it records

NB05 writes `manifest_pipeline_v1_2.csv` — one row per analysis artifact, with:

- `file` (the artifact's path)
- `hash_sha256_16` (the artifact's hash)
- `size_bytes`
- `run_id` (the run that produced it)
- `notebook_version`
- `timestamp`

### 8.2 — What "manifest" means

The manifest is the **single artifact that indexes every other artifact.** A reviewer who wants to verify "did the pipeline produce the artifacts it claims to have produced?" reads the manifest. **The manifest's rows are the claims; the artifacts on disk are the evidence.**

**The manifest is self-referential in one important way:** it does not include itself. A manifest that recorded its own hash would create a chicken-and-egg problem — the manifest's hash would depend on the manifest's contents, which would depend on the hash. **The manifest's hash is recorded in the audit log instead.** NB05's audit log records it.

### 8.3 — Where the evidence lives

The manifest is `outputs/reports/manifest_pipeline_v1_2.csv`. The manifest's own hash is in NB05's audit log. **A reviewer who wants to verify the manifest reads the audit log for its hash, then reads the manifest for the artifact hashes.**

`[ANALYST: insert the analyst's framing of the manifest's role. The manifest is a hash index; the framing is what the index makes verifiable. Every artifact in the manifest is a claim the pipeline makes about itself — the manifest is what makes the claim checkable. A reviewer who compares the manifest's hashes to the artifacts on disk is verifying the pipeline's self-claims.]`

---

## 9. What the self-assessment does **not** cover

The six components above are the FM pipeline's self-assessment. **They are not a second-line validation.** What the self-assessment does not cover:

| Not covered | Why |
|---|---|
| Whether the methodology is correct | The methodology's correctness is a validation question, not a self-assessment question. The framework document's "what this does NOT demonstrate" section is the honest statement. |
| Whether the parameters are calibrated | The parameters are analyst-specified, not calibrated. Their correctness is a production-implementation question, documented as L1–L10 in the CFR. |
| Whether the results are production-ready | The results are illustrative, not forecasts. The framework document says this explicitly. |
| Whether the framework is IFRS 9-compliant | Compliance is a determination an institution makes against its obligations. The pipeline produces evidence; the analyst produces the verdict. The claim-authorization self-check is what keeps the two separate. |

**The absence of these four is not a defect of the self-assessment** — it is what distinguishes a self-assessment from a validation. **A self-assessment answers "did the pipeline do what it said it would do?"; a validation answers "is what the pipeline did correct?"** The FM pipeline does the first; it does not do the second, and the RVP is honest about that.

`[ANALYST: insert the analyst's framing of what the self-assessment does not cover. The four items above are the boundary; the framing is why the boundary matters. A reader who expects the RVP to be a validation plan will find the boundary and be disappointed; a reader who reads the RVP's framing note will understand what the plan is and not be disappointed. The framing is what makes the boundary a feature, not a gap.]`

---

## 10. Responsibility matrix

| Role | Responsibility in self-assessment | Authority |
|---|---|---|
| IFRS 9 Modeler | Implements the six components; runs the pipeline; produces the artifacts | Implementation only |
| Validator | Reviews the six components' results; assesses the framework (MAP-STF-FM-001) | Advisory (per DCR §8) |
| Methodology Owner | Accepts the self-assessment's results; approves the framework (FM-AR-001) | Approval |
| Audit (if applicable) | Reviews the self-assessment's evidence trail | Investigation only |

**The three-line separation:**

```
Modeler (builds) → Validator (reviews) → Methodology Owner (approves)
```

**The separation is the self-assessment's governance.** The Modeler does not approve; the Validator does not build; the Methodology Owner does not build or validate. **Each role's authority is limited to its line.** This is what makes the self-assessment more than a self-report — it is a self-report that has been reviewed and approved by roles that did not produce it.

`[ANALYST: insert the analyst's framing of the responsibility matrix. The three-line separation is the governance; the framing is why it matters for a methodology demonstration. In a model validation, the separation is enforced by independence (the validator is outside the model development team). In a self-assessment, the separation is enforced by role (the Modeler builds, the Validator reviews, the Methodology Owner approves). The two are different mechanisms for the same governance principle.]`

---

## 11. Review cadence

| Field | Value |
|---|---|
| **Scheduled review of the self-assessment** | At the next MAJOR version bump of the DCR or the framework |
| **Event-triggered review** | After any change to the six components |
| **Review owner** | Methodology Owner |
| **Review inputs** | The pipeline's current state; the DCR's current state; the CFR's current state; the manifest's current state |

**The review cadence is what keeps the self-assessment current.** A self-assessment that was accurate when it was written may be inaccurate after the pipeline changes. The cadence ensures the self-assessment is reviewed at the same time the pipeline is reviewed.

`[ANALYST: insert the analyst's framing of the review cadence. The cadence is a policy; the framing is why this cadence (MAJOR version bump, not annual, not quarterly) is right for the FM project. If the cadence is tied to the DCR's change-control rhythm, state that. If it is tied to a regulatory cycle, state that. If it is a policy default, state that.]`

---

## 12. Document control

| Field | Value |
|---|---|
| **Document ID** | FM-RVP-001 |
| **Version** | 0.1 |
| **Status** | Draft — pending analyst completion of the marked sections |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | None (first version) |
| **Related documents** | FM-PDR-001 · FM-DS-001 · FM-FDL-001 · FM-MD-001 · FM-DCR-001 · FM-CFR-001 · MAP-STF-FM-001 · MAP-RP-001 |
| **Next review** | Per §11 |

---

## 13. What this document contains, and what it does not

### What it contains (from the notebook output and the FM documents)

- The framing note: this is an internal self-assessment plan, not a second-line validation plan.
- The six self-assessment components, with enforcement mechanisms and evidence.
- The responsibility matrix.
- The review cadence.
- The boundary of what the self-assessment does not cover.

### What it does not contain (the `[ANALYST: ...]` markers)

- **§3.4** — the analyst's framing of the register guards' role.
- **§4.3** — the analyst's framing of "fail loudly."
- **§5.3** — the analyst's framing of the reconciliation assertion.
- **§6.3** — the analyst's framing of the audit log's completeness.
- **§7.3** — the analyst's framing of the claim-authorization self-check.
- **§8.3** — the analyst's framing of the manifest's role.
- **§9** — the analyst's framing of what the self-assessment does not cover.
- **§10** — the analyst's framing of the responsibility matrix.
- **§11** — the analyst's framing of the review cadence.
- **§12** — the document owner's name and role.

**The document's structure and mechanism descriptions are notebook-sourced.** The components, the enforcement mechanisms, and the evidence locations are in the notebook code and the FM documents. The framing — the *why* of each component, the boundary of the self-assessment, the separation of roles — is the analyst's contribution.

**When the markers are filled, the document is complete.** It is the last of the ten FM `governance/` files. The FM `governance/` folder now has all ten documents: PDR, AS, DS, FDL, MD, RVP, DCR, CFR, AR (template), CAR. **The folder is complete.**
