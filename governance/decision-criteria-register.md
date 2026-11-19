# Decision Criteria Register (A07)

| Metadata Field | Value |
| :--- | :--- |
| **Document ID** | `FM-DCR-001` |
| **Version** | `1.0.0-planning` |
| **Freeze timestamp** | `YYYYMMDD_HHMMSS` *(literal placeholder — the register is not frozen until this is populated. The freeze must occur before any tool runs; see §2.)* |
| **Author / Owner** | `[ANALYST: name, role]` |
| **Project Scope** | IFRS 9 Expected Credit Loss (ECL) Methodology Demonstration (Public Data) |
| **Artifact Type** | Planning document — pre-registers the criteria and thresholds to be tested against, before execution. Not a run record; measured results belong to A11. |
| **Regulatory Alignment References** | IFRS 9, OSFI E-23, SR 26-2, ECOA *(illustrative/defensibility alignment; not a compliance certification)* |
| **Cross-Referenced Artifacts** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03), `FM-FDL-001` (A04), `FM-MD-001` (A05), `FM-RVP-001` (A06), `FM-CFR-001`, `MAP-STF-FM-001`, A11 (Results), A13 (Interpretation Record) |

> **Governance & Scope Notice:** This register pre-registers the criteria, thresholds, and fail conditions the IFRS 9 ECL demonstration is tested against. It is frozen before execution and is not modified by results. It is a planning artifact: it defines the standard, not the outcome. Measured results and verdicts are recorded in A11 (Results); interpretation is in A13 and `MAP-STF-FM-001`. This is not a production model specification and not an independent validation report.

> **Provenance note (register synthesis):** This register is a merge of two prior A07 sources. **Source A** is the fuller DCR body (§1–§8: guard design, change control, authority, reproducibility). **Source B** is the AP-01–AP-10 pre-registration table with investigate bands, fail conditions, anti-gaming provisions, and the AP-02 v1.0→v1.1 disclosure. Where the two sources disagree on a threshold, both values are shown with a provenance marker; the adopted value is **not** chosen by this document and is marked `[ADOPTED: A | B — ANALYST: choose]`. The merge rule: Source B's three-column threshold discipline (threshold / investigate band / fail condition) is canonical; Source A's two-tier structure (target / failure) is represented within it.

> **Status-line convention:** As in A04 and A05, each substantive section ends with a status line:
> - `Decided` — the decision is made and recorded here.
> - `Deferred to [artifact] — owner: [role]` — intentionally not decided here.
> - `Open — decision needed by [milestone]` — no decision exists yet and one is required by a stated milestone.

---

## 1. Criterion catalogue — pre-registered table (Source B, canonical form)

This is the register's front door. It is frozen before execution. The three-column threshold discipline is canonical: **threshold** is the target, **investigate band** is the range that triggers investigation (not failure), and **fail condition** is the range outside which the criterion fails.

| ID | Criterion | Threshold | Critical? | Investigate band | Fail condition | Provenance |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **AP-01** | PD calibration | ±0.06 pp (95% CI) | Yes | ±0.08 pp | > ±0.08 pp | Source B only |
| **AP-02** | LGD benchmark | 40–60% | Yes | `[ADOPTED: A \| B — ANALYST: choose]` — A: 35–65%; B: 35–65% | `[ADOPTED: A \| B — ANALYST: choose]` — A: outside 30–70%; B: outside 35–65% | A + B (conflict) |
| **AP-02a** | Constructed LGD disclosure | Constructed LGD disclosed as constructed; if empirical LGD used, benchmark to 40–60% | Yes | — | — | Source B only |
| **AP-03** | EAD ratio | 60–75% | Yes | `[ADOPTED: A \| B — ANALYST: choose]` — A: 55–80%; B: 55–80% | `[ADOPTED: A \| B — ANALYST: choose]` — A: outside 50–85%; B: outside 55–80% | A + B (conflict) |
| **AP-04** | ECL ratio of EAD | 1–3% | Yes | `[ADOPTED: A \| B — ANALYST: choose]` — A: 0.8–3.5%; B: 0.8–3.5% | `[ADOPTED: A \| B — ANALYST: choose]` — A: outside 0.5–5%; B: outside 0.8–3.5% | A + B (conflict) |
| **AP-05** | Scenario severity | ≥ 1.5× historical | Yes | 1.3–1.5× | < 1.3× | A + B (consistent) |
| **AP-06** | Segment monotonicity | Monotone | No | — | — | Source B only |
| **AP-07** | Stage distribution | Stage 2 > 0% | No | — | — | Source B only (reasonableness check, not a quality criterion — see §3.2) |
| **AP-08** | Fair lending | Pass or documented | No | — | — | Source B only |
| **AP-09** | Reproducibility | Within tolerance | No | — | — | Source B only |
| **AP-10** | Documentation | 100% decisions | No | — | — | Source B only |

> **Note on AP-02a:** Source B splits AP-02 into the original benchmark (v1.0, retained) and a v1.1 disclosure criterion (AP-02a). Both are retained here. See §3.1.

**Status:** `Open — decision needed by register freeze` for the three `[ADOPTED]` threshold conflicts; `Decided` for the criterion set and structure.

---

## 2. Freeze and anti-gaming provision (Source B)

The register is **frozen before any tool runs**. The freeze timestamp is recorded in the metadata block (`YYYYMMDD_HHMMSS` placeholder until populated).

**Anti-gaming rule (verbatim from Source B):**

> "The register is frozen before any tool runs. Any change to a criterion after freeze requires a version bump and disclosure. The change must be reported alongside the original criterion."

**What this requires:**
- A criterion may not be changed after freeze without a version bump.
- The change must be disclosed **alongside** the original criterion — the original is not erased.
- The register must show both the pre-freeze criterion and the post-freeze change.

**Why it exists:** a post-hoc correlation scan or a post-hoc benchmark reclassification cannot distinguish a true standard from a tuned one. Freezing the criteria before inspection is what makes a failure meaningful. A register that reports a failure as a failure is a contract; a register that reclassifies the benchmark is a document. The worked example in §3.1 is the test of whether this provision is real.

**Status:** `Decided`

---

## 3. Worked example — AP-02 failure disclosure (Source B, carried as-is)

### 3.1 — AP-02 v1.0 → v1.1

This is the register's worked example of the anti-gaming provision in action. It is carried verbatim from Source B. It is **not** a verdict on the current run — the measured AP-02 value for this run is marked `[DISCREPANCY — see A09]` in §4 and is not resolved here.

> **AP-02 (v1.0):** LGD benchmark 40–60%. Result: 14.18%. Status: **FAILED**.
>
> **Version 1.1 disclosure:** the benchmark was drawn from empirical loss severity on U.S. mortgage books and does not apply to a constructed LGD that is a function of PD. The benchmark is retained in v1.0 as the original criterion; v1.1 adds AP-02a (see below). The failure is reported as a failure, not reclassified.
>
> **AP-02a (v1.1):** Constructed LGD must be disclosed as constructed; if an empirical LGD is used, it must be benchmarked against 40–60%. Status: N/A for this project, because the LGD is constructed.

**What this example demonstrates:** the earlier draft reported the AP-02 failure and then called the benchmark "inappropriate" — which is post-hoc rationalization. The revised entry reports the criterion as **failed**, bumps the version, and discloses why the benchmark does not apply **alongside the original criterion**. The original 40–60% benchmark is retained, not erased.

**Status:** `Decided` (carried as the worked example); the current-run measured value is `Open` — see §4.

### 3.2 — Weak and tautological criteria: reframed, not deleted

Source B identifies criteria in the earlier draft that do not test quality:
- **"Stage 2 > 0%"** — retained as **AP-07 (reasonableness check)**, not a quality criterion. A non-degenerate staging is a precondition, not evidence of quality.
- **"Explainability: SHAP available"** — replaced by **AP-10 (documentation)**: the check is that explainability evidence *exists in A13*, not that a particular library was used.

**Status:** `Decided`

---

## 4. Results against criteria (planning posture)

This register is a planning artifact. This section records the **planned** measured-value locations and the known discrepancies that must be closed before a run verdict can be recorded. It does not record run verdicts.

| ID | Measured value source | This-run status |
| :--- | :--- | :--- |
| AP-01 | A11 / `MAP-STF-FM-001` | `Deferred to A11` |
| AP-02 | A11 / DCR §4 | `[DISCREPANCY — see A09]` (14.18% vs 44.28%; see §3.1 and A09 item) |
| AP-02a | A11 / A13 | `Deferred to A11` |
| AP-03 | A11 / DCR §4 | `Deferred to A11` (expected to fire on young portfolio per A05 §4.2 / A03) |
| AP-04 | A11 / DCR §4 | `Deferred to A11` |
| AP-05 | A11 / DCR §4 | `Deferred to A11` |
| AP-06 | A11 | `Deferred to A11` |
| AP-07 | A11 / DCR §4 | `Deferred to A11` |
| AP-08 | A11 / NB04 | `Deferred to A11` |
| AP-09 | A10 / reproducibility record | `Deferred to A10` |
| AP-10 | A13 | `Deferred to A13` |

> **On AP-02:** Source A reported 44.28% PASS; Source B reported 14.18% FAILED. Both cannot describe the same run. The discrepancy is the open A09 item. This register does not pick a value or a verdict.

> **On AP-03:** the expected firing on a young portfolio is a planned expectation from A05 §4.2, not a measured result. The measured EAD ratio is recorded in A11.

**Status:** `Open — decision needed by A11 finalization` (all measured values); `Open` for the AP-02 discrepancy.

---

## 5. Guard design — two-tier and three-column

The register uses the **three-column discipline** (threshold / investigate band / fail condition) as canonical (Source B), with Source A's two-tier structure (target band / failure band) represented within it:
- **Threshold** ≈ Source A's "target band."
- **Investigate band** = range outside threshold that triggers investigation (WARNING).
- **Fail condition** = range outside which the criterion fails (ERROR).

| ID | Threshold | Investigate band | Fail condition | Action |
| :--- | :--- | :--- | :--- | :--- |
| AP-02 | 40–60% | `[ADOPTED: A \| B]` | `[ADOPTED: A \| B]` | WARNING outside threshold; ERROR outside fail condition |
| AP-03 | 60–75% | `[ADOPTED: A \| B]` | `[ADOPTED: A \| B]` | WARNING outside threshold; ERROR outside fail condition |
| AP-04 | 1–3% | `[ADOPTED: A \| B]` | `[ADOPTED: A \| B]` | WARNING outside threshold; ERROR outside fail condition |
| AP-05 | ≥1.5× | 1.3–1.5× | <1.3× | WARNING in investigate band; ERROR outside |

**Critical point:** guards fire on thresholds, not on analyst judgment. A threshold change is a register change (see §7), not a code change. A guard that fires without a corresponding register entry is a code bug; a register entry that does not fire is a data finding.

**Status:** `Open — decision needed by register freeze` for the `[ADOPTED]` cells; `Decided` for the guard design.

---

## 6. Verdict record — status bands

The register uses three verdict states per criterion:

| Verdict | Meaning | Action |
| :--- | :--- | :--- |
| `PASS` | Within threshold | No action |
| `OUTSIDE THRESHOLD` | Within investigate band but outside threshold | Document; investigate |
| `FAILED` | Outside fail condition | Document; escalate; version-bump disclosure if the criterion is reclassified |

**On this register's planning posture:** no verdicts are recorded in this artifact. Verdicts are recorded in A11. The AP-02 worked example in §3.1 records a historical verdict (`FAILED`, v1.0) as a demonstration of the anti-gaming provision, not as this run's verdict.

**Status:** `Decided`

---

## 7. Change control and versioning (Source A, retained)

### 7.1 — Change control rules

| Change type | Version bump | Requires | Example |
| :--- | :--- | :--- | :--- |
| Parameter bound change (e.g., PD cap 0.20 → 0.25) | MAJOR | Methodology Owner approval; full re-run | Changing §7.2 PD cap |
| Register guard threshold change | MAJOR | Methodology Owner approval; re-run + disclosure | Changing AP-03 fail condition |
| Scenario weight change | MAJOR | Methodology Owner approval; re-run | Changing baseline from 45% to 50% |
| Adding a new criterion | MINOR | Validator approval; documented | Adding AP-06 |
| Clarifying wording | PATCH | No approval; documented | Fixing a typo |
| Structural reorganization (no threshold change) | MINOR | No approval; documented | §3/§4/§5 reorganization |

**Rule:** any change that affects a parameter or a guard's behaviour is a MAJOR version bump. A structural reorganization that does not change any threshold is MINOR. Wording clarification is PATCH. The Fannie Mae register is sensitive to MAJOR bumps because parameters are frozen in code — changing them requires a full pipeline re-run.

### 7.2 — Version history

| Version | Date | Change | Type |
| :--- | :--- | :--- | :--- |
| 1.0.0-planning | `[ANALYST: date]` | Register synthesis of Source A + Source B; three-column discipline adopted; freeze timestamp added; AP-02 worked example carried | — |
| (v1.0 → v1.1 AP-02 disclosure) | — | AP-02 reported FAILED (14.18%); benchmark retained; AP-02a added | MINOR (disclosure) |

**Status:** `Decided`

---

## 8. Authority and approval (Source A, retained)

| Change | Proposer | Approver | Rationale required |
| :--- | :--- | :--- | :--- |
| PD/LGD bounds | IFRS 9 Modeler | Methodology Owner | Yes |
| Register guard thresholds | Modeler | Methodology Owner | Yes |
| Scenario weights | Modeler | Methodology Owner | Yes |
| New criterion | Validator | Methodology Owner | Yes |
| New parameter | Modeler | Methodology Owner | Yes |

The Methodology Owner is the sole authority for register changes. The freeze is valid only if the freeze timestamp in the metadata block is populated before execution.

**Status:** `Decided`

---

## 9. Completeness and coverage check (self-assessment)

Per criterion: is it **defined**, **measured this run**, and **verdict recorded**? This table converts "is A07 complete?" into a check the register answers about itself.

| ID | Defined? | Measured? | Verdict recorded? | Open item |
| :--- | :--- | :--- | :--- | :--- |
| AP-01 | Yes (B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-02 | Yes (A+B; conflict) | `[DISCREPANCY — A09]` | `Open` | Threshold adoption + 14.18/44.28 |
| AP-02a | Yes (B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-03 | Yes (A+B; conflict) | `Deferred to A11` | `Deferred to A11` | Threshold adoption |
| AP-04 | Yes (A+B; conflict) | `Deferred to A11` | `Deferred to A11` | Threshold adoption |
| AP-05 | Yes (A+B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-06 | Yes (B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-07 | Yes (B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-08 | Yes (B) | `Deferred to A11` | `Deferred to A11` | — |
| AP-09 | Yes (B) | `Deferred to A10` | `Deferred to A10` | — |
| AP-10 | Yes (B) | `Deferred to A13` | `Deferred to A13` | — |

**Register-level open items:** (i) three `[ADOPTED: A | B]` threshold conflicts; (ii) AP-02 measured value 14.18% vs 44.28% (A09 item); (iii) freeze timestamp not yet populated.

**Status:** `Open — decision needed by register freeze` for (i) and (iii); `Open` for (ii).

---

## 10. Provenance appendix — retired cross-project comparison (Source A)

Source A contained comparison paragraphs between this register and a Lending Club register (different criterion families; LC-G1–LC-G6 vs AP-02–AP-04; unrelated numbering). Those paragraphs are retained here as provenance, not as part of the register contract.

- **Lending Club register** is organized around model performance criteria (AUC, KS, DI, HL).
- **This register** is organized around parameter defensibility (are the values justified?) and guard behaviour (do the checks fire as expected?).
- The two registers' guard numbering schemes are unrelated; AP-02 here is not AP-02 there.

**Status:** `Decided` (appendix; not part of the frozen criterion set)

---

## 11. Document control

| Field | Value |
| :--- | :--- |
| **Document ID** | `FM-DCR-001` |
| **Version** | `1.0.0-planning` |
| **Status** | Draft — planning artifact; pending register freeze, threshold adoption, and AP-02 discrepancy closure |
| **Owner** | `[ANALYST: name, role]` |
| **Supersedes** | Source A (DCR §1–§8 body) and Source B (AP-01–AP-10 pre-registration table), merged |
| **Related documents** | `FM-PDR-001` (A01), `FM-AS-001` (A02), `MAP-DS-FM-001` (A03), `FM-FDL-001` (A04), `FM-MD-001` (A05), `FM-RVP-001` (A06), `FM-CFR-001`, `MAP-STF-FM-001`, A11, A13 |
| **Next review** | At register freeze, or upon closure of the AP-02 discrepancy |
