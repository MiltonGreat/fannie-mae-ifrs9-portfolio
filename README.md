# Fannie Mae IFRS 9 Compliance Framework

## Portfolio Project: Credit Risk Modeling & Regulatory Compliance

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![IFRS 9](https://img.shields.io/badge/IFRS-9-orange.svg)](https://www.ifrs.org/)
[![OSFI](https://img.shields.io/badge/OSFI-E--23-red.svg)](https://www.osfi-bsif.gc.ca/)
[![SR 11-7](https://img.shields.io/badge/SR--11-7-purple.svg)](https://www.federalreserve.gov/)
[![ECOA](https://img.shields.io/badge/ECOA-Fair%20Lending-blue.svg)](https://www.ecoa.gov/)

---

## 📋 Executive Summary

This project demonstrates a complete **IFRS 9 Expected Credit Loss (ECL) framework** applied to a Fannie Mae mortgage portfolio. It covers the full lifecycle from data preparation to regulatory reporting, with a strong emphasis on **Governance, Risk, and Compliance (GRC)**.

### Key Findings

| Metric | Value | Assessment |
|--------|-------|------------|
| **Default Rate** | 0.94% | ✅ Within expected range |
| **Unconditional PD** | 5.76% | ✅ Enhanced estimate |
| **Average LGD** | 14.18% | ✅ Collateral segmentation applied |
| **Average EAD** | $236,040 | ✅ Prepayment/curtailment assumptions |
| **Probability-Weighted ECL** | $4,597 | ✅ IFRS 9 compliant |
| **Capital Impact Ratio** | 0.01% | ✅ Well-capitalized |
| **Fairness Testing** | 0/3 Pass | ⚠️ Remediation required |

### Regulatory Compliance

| Regulation | Status | Evidence |
|------------|--------|----------|
| **IFRS 9** | ✅ Compliant | 12-month PD, stage classification, probability-weighted ECL |
| **OSFI E-23** | ✅ Compliant | Missing indicators, data quality, fairness testing |
| **SR 11-7** | ✅ Compliant | Model validation, ROC curve, outcomes analysis |
| **ECOA** | ⚠️ Partial | Disparate impact analysis; remediation recommended |
| **Basel III** | ✅ Compliant | Capital adequacy assessment |

---

## 🎯 Project Objectives

1. **Demonstrate IFRS 9 compliance** methodology using real mortgage data
2. **Build a complete ECL framework** from data to reporting
3. **Document all decisions** with rationale and audit trail
4. **Assess capital adequacy** under stress scenarios
5. **Test for fairness** across protected attributes
6. **Generate audit-ready reports** for regulatory submission

---

## 📓 Notebooks Overview

| Notebook | Focus | Key Outputs | Regulatory Link |
|----------|-------|-------------|-----------------|
| **01_Data_Loading** | Data quality & cleaning | Cleaned dataset, 33 missing indicators | OSFI E-23 |
| **02_EDA** | Risk factor analysis | Risk drivers, correlations | SR 11-7 |
| **03_Calibration** | PD/LGD/EAD/ECL | IFRS 9 parameters | IFRS 9 |
| **04_Stress_Testing** | Scenarios & capital | Scenario ECL, fairness testing | IFRS 9, ECOA |
| **05_Reporting** | Dashboard & report | Compliance report, decision log | All |
| **06_PowerBI_Export** | Data export | Power BI tables | Reporting |

---

## 📊 Key Results

### IFRS 9 Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Unconditional PD** | 0.94% | Base probability of default |
| **Enhanced PD** | 5.76% | With non-linear risk relationships |
| **Average LGD** | 14.18% | Collateral segmentation applied |
| **Average EAD** | $236,040 | Prepayment/curtailment assumptions |
| **Portfolio ECL** | $303 | Weighted average by stage |
| **Probability-Weighted ECL** | $4,597 | IFRS 9 reported ECL |

### Scenario Analysis

| Scenario | PD | LGD | ECL | Weight | Weighted ECL |
|----------|-----|-----|-----|--------|--------------|
| **Baseline** | 5.76% | 14.18% | $1,863 | 45% | $838 |
| **Adverse** | 10.37% | 17.02% | $4,024 | 30% | $1,207 |
| **Severely Adverse** | 17.28% | 21.27% | $8,383 | 15% | $1,257 |
| **Tail Risk** | 20.00% | 28.37% | $12,938 | 10% | $1,294 |

**Total Probability-Weighted ECL: $4,597**

### Capital Adequacy

| Metric | Value |
|--------|-------|
| **Total Assets** | $1,000,000,000 |
| **CET1 Capital** | $45,000,000 |
| **ECL Impact** | $4,597 |
| **Capital Impact Ratio** | 0.01% |
| **Assessment** | ✅ Adequate |

### Fairness Testing

| Attribute | Disparate Impact | 4/5ths Rule | Status |
|-----------|------------------|-------------|--------|
| **FICO_BOR** | 0.00% | ❌ Fail | Data sparsity issue |
| **PROP_TYPE** | 31.00% | ❌ Fail | Mobile Homes higher risk |
| **OCCU_STAT** | 48.17% | ❌ Fail | Primary residences higher risk |

> **Note:** The FICO_BOR failure is driven by data sparsity. PROP_TYPE and OCCU_STAT failures reflect legitimate risk differences, not discrimination. Remediation recommended.

---

## 🎯 GRC Deliverables

### Governance Artifacts

| Artifact | Location | Description |
|----------|----------|-------------|
| **Decision Log** | `governance/decision_log.csv` | 18 documented decisions with rationale |
| **Regulatory Mapping** | `governance/regulatory_mapping.csv` | 5 regulations mapped |
| **Model Inventory** | `governance/model_inventory.csv` | Model documentation |

### Reports

| Report | Location | Description |
|--------|----------|-------------|
| **Executive Summary** | `reports/executive_summary.md` | Key findings for leadership |
| **Findings Register** | `reports/findings_register.md` | All findings with severity |
| **Remediation Plan** | `reports/remediation_plan.md` | Action plan with owners |

### Compliance Documentation

| Document | Location | Description |
|----------|----------|-------------|
| **Methodology** | `docs/methodology.md` | Detailed methodology |
| **Regulatory Compliance** | `docs/regulatory_compliance.md` | Compliance mapping |
| **Interpretation Guide** | `docs/interpretation_guide.md` | How to interpret results |

---

## 📋 Key Decisions

| ID | Category | Decision | Rationale |
|----|----------|----------|-----------|
| D001 | Data Quality | Random sampling (100,000 rows) | Prevents temporal bias |
| D002 | Data Quality | Median imputation | Robust to outliers |
| D003 | Feature Engineering | LTV threshold: 80% | PMI requirement |
| D004 | Feature Engineering | FICO bands: 680/720/760 | Industry standard |
| D005 | PD Calibration | 12-month observation window | IFRS 9 requirement |
| D006 | Scenario Design | 4 scenarios including Tail Risk | Comprehensive stress testing |
| D007 | Scenario Design | Weights: 45/30/15/10 | Probability-weighted ECL |
| D008 | Capital Assessment | CET1 ratio: 4.5% | Basel III minimum |
| D009 | Fairness Testing | 4/5ths rule (80%) | ECOA standard |

*Full decision log available in `governance/decision_log.csv`*

---

## 🚀 Getting Started

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
---
# Data Directory

## Data Source

This project uses the Fannie Mae Single-Family Loan Performance dataset.

### Download Instructions

**Visit Kaggle.com to find the Fannie Mae Data:**
   - Go to: [https://www.fanniemae.com/research-and-insights/data](https://www.kaggle.com/datasets/pranay07/fanne-mae-loan-performance-data)
