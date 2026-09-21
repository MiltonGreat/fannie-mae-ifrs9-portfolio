# Fannie Mae IFRS 9 Compliance Framework

## Portfolio Project: Credit Risk Modeling & Regulatory Compliance

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![IFRS 9](https://img.shields.io/badge/IFRS-9-orange.svg)](https://www.ifrs.org/)
[![OSFI](https://img.shields.io/badge/OSFI-E--23-red.svg)](https://www.osfi-bsif.gc.ca/)
[![SR 11-7](https://img.shields.io/badge/SR--11--7-purple.svg)](https://www.federalreserve.gov/)
[![ECOA](https://img.shields.io/badge/ECOA-Fair%20Lending-blue.svg)](https://www.ecoa.gov/)

---

## Executive Summary

This project demonstrates an **IFRS 9 Expected Credit Loss (ECL) framework**
applied to a Fannie Mae mortgage portfolio. It covers the full lifecycle from
data preparation to regulatory reporting, with an emphasis on **Governance,
Risk, and Compliance (GRC)**.

The work is a **methodology demonstration**, not a production implementation.
It shows how IFRS 9 concepts are implemented in code, how decisions are
documented for audit, and where a real bank's production environment would
need to substitute proprietary data and calibration.

### Key Results

| Metric | Value | Notes |
|--------|-------|-------|
| **Portfolio size** | 100,000 loans | Random sample from Fannie Mae dataset |
| **Default rate (observed)** | 0.94% | Backward-looking frequency |
| **PD (calibrated)** | 5.76% | Forward-looking probability estimate |
| **Average LGD** | 14.18% | Collateral segmentation applied |
| **Average EAD (per loan)** | $236,040 | After prepayment/curtailment |
| **Probability-weighted ECL (per loan)** | $4,597 | Across 4 stress scenarios |
| **Portfolio ECL (weighted)** | ~$460M | Per-loan ECL × portfolio size |
| **Capital impact ratio (portfolio)** | ~1,000% | ECL vs. CET1 under stress |

### Key Findings

**What worked well:**

- IFRS 9 three-stage classification implemented, with a documented proxy for
  SICR detection where the data lacks observed current credit characteristics
- Probability-weighted ECL calculated across four stress scenarios
- Decision log documents 20+ methodological choices with rationale and
  regulatory references
- Complete audit trail — all runs timestamped and logged

**What requires documented limitations:**

- **SICR detection is a proxy**, not true SICR, because the dataset contains
  no observed current credit characteristics to compare against origination
- **Fair lending testing was not performed** — the dataset contains no
  protected-class attributes (race, ethnicity, sex, marital status) or
  usable geographic proxies
- **Capital adequacy is inadequate** under portfolio-aggregate ECL, contrary
  to earlier per-loan comparisons
- **PD-LGD correlation is constructed**, not calibrated from recovery data

---

## Regulatory Compliance

Each framework is assessed against what the project actually demonstrates,
not what it would require in production.

| Regulation | Status | Notes |
|------------|--------|-------|
| **IFRS 9** | Compliant with limitations | ECL measurement, staging, and probability weighting implemented. SICR uses proxy. |
| **OSFI E-23** | Compliant with limitations | Data quality and documentation practices followed. Fair lending testing not performed. |
| **SR 11-7** | Compliant | Model validation, ROC curve, and outcomes analysis implemented. |
| **ECOA** | Not assessed | Dataset lacks protected-class attributes. Cannot be tested on this data. |
| **Basel III** | Compliant with limitations | Capital adequacy framework implemented. Result shows inadequate coverage under stress. |

A full regulatory mapping — including the specific requirements addressed,
the evidence, and the gaps — is in `governance/regulatory_mapping.csv` and
`reports/regulatory_mapping_enhanced.txt`.

---

## Project Objectives

1. Demonstrate IFRS 9 ECL methodology using publicly available mortgage data
2. Build an end-to-end pipeline from raw data to regulatory reporting
3. Document every methodological decision with rationale and audit trail
4. Assess capital adequacy under stress scenarios
5. Implement fair lending testing where protected attributes are available
6. Generate audit-ready artifacts suitable for internal review

---

## Notebooks Overview

| Notebook | Focus | Key Outputs | Regulatory Link |
|----------|-------|-------------|-----------------|
| **01 Data Loading** | Quality & cleaning | Cleaned dataset, missing indicators, assumptions register | OSFI E-23 |
| **02 EDA** | Risk factor analysis | Risk drivers, correlations, vintage patterns | SR 11-7 |
| **03 Calibration** | PD/LGD/EAD calibration | IFRS 9 parameters, stage classification | IFRS 9 |
| **04 Stress Testing** | Scenarios & capital | Scenario ECL, capital adequacy, fairness | IFRS 9, ECOA |
| **05 Reporting** | Dashboard & report | Compliance report, decision log, regulatory mapping | All |
| **06 Power BI Export** | Data export | Fact and dimension tables | Reporting |

Run the notebooks in order — each depends on artifacts produced by the
preceding ones.

---

## IFRS 9 Parameters

All figures are per loan unless labeled otherwise.

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Default rate (observed)** | 0.94% | Backward-looking frequency in sample |
| **PD (calibrated)** | 5.76% | Forward-looking probability, averaged across loans |
| **Average LGD** | 14.18% | After collateral segmentation by property type |
| **Average EAD** | $236,040 | After prepayment and curtailment adjustments |
| **ECL (unstressed, per loan)** | $308 | PD × LGD × EAD |
| **Probability-weighted ECL (per loan)** | $4,597 | Across four scenarios with weights |

**IFRS 9 stage distribution:**

| Stage | Share | ECL Basis |
|-------|-------|-----------|
| Stage 1 | ~89% | 12-month ECL |
| Stage 2 | ~10% | Lifetime ECL (proxy SICR) |
| Stage 3 | ~0.9% | Lifetime ECL (credit-impaired) |

Stage 2 population is populated using a proxy — the highest-risk decile
of non-defaulted loans — because the dataset lacks the current credit
characteristics required for true SICR detection.

---

## Scenario Analysis

| Scenario | PD | LGD | ECL (per loan) | Weight | Weighted ECL |
|----------|-----|-----|----------------|--------|--------------|
| **Baseline** | 5.76% | 14.18% | $1,863 | 45% | $838 |
| **Adverse** | 10.37% | 17.02% | $4,024 | 30% | $1,207 |
| **Severely Adverse** | 17.28% | 21.27% | $8,383 | 15% | $1,257 |
| **Tail Risk** | 20.00% | 28.37% | $12,938 | 10% | $1,294 |

**Probability-weighted ECL (per loan): $4,597**

The Tail Risk scenario's PD is capped at 20% (the PD cap in the config).
Without the cap it would be 25.9%; the cap mutes the convexity effect
between the two most severe scenarios.

---

## Capital Adequacy

| Metric | Value |
|--------|-------|
| **Total assets (assumed)** | $1,000,000,000 |
| **CET1 ratio** | 4.50% (Basel III minimum) |
| **CET1 capital** | $45,000,000 |
| **Portfolio ECL (weighted)** | ~$460,000,000 |
| **Capital impact ratio** | ~1,000% |
| **Assessment** | **Inadequate** under portfolio-aggregate stress |

**Interpretation:** The illustrative CET1 capital of $45M is far below what a
100,000-loan mortgage portfolio would need to absorb its stressed ECL. The
test is honest about this — the impact ratio is over 1,000%, which means the
assumed capital would be exhausted nearly ten times over.

For a production version, the assumed total assets would be calibrated to the
actual portfolio, and the test would reflect the institution's real capital
position.

---

## Fairness Testing

| Assessment | Status |
|------------|--------|
| **Fair lending testing** | **Not performed** |
| **Reason** | Dataset contains no protected-class attributes or usable geographic proxies |
| **Protected attributes checked for** | race, ethnicity, sex, gender, marital status, national origin, religion, age band, and geographic fields (state, region, census tract, ZIP) |
| **Result** | No matching fields found |

**What this means:**

Fair lending testing under ECOA (US), OSFI E-23 s.6.0 (Canada), and equivalent
regimes requires testing on protected classes. This dataset does not contain
them, and fair lending compliance cannot be verified from it.

**What the notebook does not do:**

The notebook does not test disparate impact on credit risk drivers (FICO,
property type, occupancy status). Those are legitimate underwriting variables,
not protected classes, and testing them under a "fairness" label would produce
false positives — differences that are expected and legal.

**What a production fair lending assessment would require:**

Data containing protected attributes or geographic identifiers (ZIP code,
census tract). For US mortgage portfolios, this is available through HMDA
reporting.

---

## GRC Deliverables

### Governance Artifacts

| Artifact | Location | Description |
|----------|----------|-------------|
| **Decision Log** | `outputs/decision_log_enhanced.txt` | 20 decisions with rationale, regulatory reference, and alternatives |
| **Regulatory Mapping** | `outputs/regulatory_mapping_enhanced.txt` | Requirements mapped, with status and gaps |
| **Assumptions Register** | `data/assumptions_register.csv` | Central register of all thresholds and parameters |

### Reports

| Report | Location | Description |
|--------|----------|-------------|
| **Compliance Report** | `outputs/ifrs9_compliance_report_enhanced.txt` | 10-section report with results, methodology, and limitations |
| **Calibration Summary** | `outputs/pd_lgd_ead_calibration_summary_v2_1.txt` | PD/LGD/EAD calibration results |
| **Stress Testing Summary** | `outputs/stress_testing_summary.txt` | Scenario results and capital adequacy |
| **Final Summary** | `outputs/final_summary_enhanced.txt` | Project-level summary |

### Audit Trail

Every notebook writes a timestamped log to `logs/`. Each run has a unique
`RUN_ID` (timestamp-based), and the log records every significant decision
with its time of execution.

---

## Key Decisions

| ID | Category | Decision | Rationale |
|----|----------|----------|-----------|
| D001 | Data Quality | Random sampling (100,000 rows) | Prevents temporal bias |
| D002 | Data Quality | Median imputation | Robust to outliers |
| D003 | Feature Engineering | LTV threshold: 80% | Aligns with PMI requirement |
| D004 | Feature Engineering | FICO bands: 680/720/760 | Industry standard for risk segmentation |
| D005 | PD Calibration | 12-month observation window | IFRS 9 requirement for Stage 1 |
| D006 | PD Calibration | PIT PD (not TTC) | IFRS 9 requires current conditions |
| D007 | SICR Implementation | Proxy approach (top-decile risk) | Dataset lacks current credit characteristics |
| D008 | Scenario Design | 4 scenarios including Tail Risk | Comprehensive stress testing |
| D009 | Scenario Weights | 45/30/15/10 | Probability-weighted ECL |
| D010 | Capital Assessment | Portfolio-aggregate ECL | Correct scaling for CET1 comparison |
| D011 | Fairness Testing | Protected classes only | Risk drivers are not protected classes |
| D012 | Reporting | Static dashboard + Power BI export | Regulatory submission + interactive exploration |

*Full decision log available in `outputs/decision_log_enhanced.txt`*

---

## Getting Started

### Prerequisites

- Python 3.8+
- Jupyter Notebook or JupyterLab
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/fannie-mae-ifrs9-portfolio.git
cd fannie-mae-ifrs9-portfolio

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
