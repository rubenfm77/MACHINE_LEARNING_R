# Employee Attrition Prediction — ML Capstone in R

[![R](https://img.shields.io/badge/R-4.2+-blue?logo=r)](https://www.r-project.org/)
[![H2O](https://img.shields.io/badge/H2O-AutoML-yellow)](https://h2o.ai)
[![Dataset](https://img.shields.io/badge/Dataset-IBM%20HR%20(Kaggle)-lightgrey)](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)

> End-to-end capstone project predicting employee attrition on the IBM HR dataset (1,470 employees, 35 variables) — full EDA, class balancing, model bake-off across four algorithm families, and a Power BI dashboard translating findings into actionable HR intelligence.

---

## Repository Contents

```
MACHINE_LEARNING_R/
├── code/
│   ├── Ruben_Fernandez_Martinez_TFM.Rmd   # Full R notebook (EDA + modelling)
│   └── TFM_presentation.pptx               # Business-facing PowerPoint
└── README.md
```

The companion **Power BI dashboard** (HR Attrition) lives in the [POWER-BI repo](https://github.com/rubenfm77/POWER-BI) and was built directly from the conclusions of this analysis.

---

## The Problem

Employee turnover costs organisations an average of **1–2× annual salary** per departure. The IBM HR dataset (16.1% attrition rate — above the industry benchmark of 10%) provides a realistic testbed for building an early-warning model that identifies at-risk profiles before they resign.

---

## Approach

### 1. Exploratory Data Analysis

Full EDA across all 35 variables before any modelling:

- **Attrition by department:** Sales highest rate (5.75%), R&D highest absolute count
- **Attrition by role:** Lab Technicians, Sales Executives, Research Scientists, and Sales Reps are the four most at-risk job functions
- **Attrition by marital status:** Singles have ~3× higher attrition than divorced employees
- **Overtime effect:** Employees doing overtime leave at a significantly higher rate than those who don't
- **Salary correlation:** Employees with lower hourly rates and monthly income consistently show higher attrition — combining low pay with poor work-life balance is the highest-risk combination
- **Satisfaction scores:** Employees who leave score lower on both job satisfaction and environment satisfaction; the gap is most pronounced among low-salary earners
- **Tenure patterns:** Employees with 0–1 years at the company (and 1 previous employer) have the highest early-exit rate, suggesting onboarding and early-career support gaps

### 2. Feature Engineering & Preprocessing

- Target variable `Attrition` recoded to binary (1 = Yes, 0 = No)
- Min-max normalisation applied to high-range numeric variables (MonthlyIncome, MonthlyRate, etc.) to prevent gradient distortion
- High-cardinality categorical variables converted to factors for H2O compatibility
- Low-information variables removed: `Over18`, `EmployeeCount`, `StandardHours` (zero variance), `EmployeeNumber` (ID only)
- Class imbalance addressed via `balance_classes = TRUE` in H2O (14.2% positive rate)
- Correlation analysis: `Age` and `JobLevel` highly correlated (removed redundancy); `PerformanceRating` and `PercentSalaryHike` correlated

### 3. Model Bake-off

Four algorithm families compared on AUC (Area Under ROC Curve):

| Model | AUC | Notes |
|---|---|---|
| **H2O AutoML — StackedEnsemble** | **0.84** | Best; combines GBM + DL + RF base learners |
| H2O Deep Learning | 0.82 | Strong on numeric-heavy features |
| H2O Gradient Boosting Machine | 0.78 | Consistent; fast to tune |
| H2O Random Forest | 0.77 | Good baseline; interpretable feature importance |

All models used 5-fold cross-validation and class balancing.

---

## Key Dashboard Visuals

**Power BI HR Dashboard — Page 1: Attrition Overview**

![HR Dashboard 1](https://raw.githubusercontent.com/rubenfm77/POWER-BI/main/HRDash_1.jpg)

**Power BI HR Dashboard — Page 2: Risk Factors**

![HR Dashboard 2](https://raw.githubusercontent.com/rubenfm77/POWER-BI/main/HRDash_2.jpg)

---

## Conclusions

**Model performance and what it means in practice**

The best model (H2O StackedEnsemble, AUC = 0.84) correctly identifies at-risk employees at a rate far above random. An AUC of 0.84 means that when the model is asked to compare one employee who eventually leaves against one who stays, it ranks the leaver higher 84% of the time. In a workforce of 1,470, this allows HR to focus retention effort on the top 20% highest-risk profiles — a manageable action list.

**The drivers that matter most**

All three model families agree on the core signal (in rough order of importance):

1. **Overtime** — the single strongest predictor across all models. Employees doing overtime leave at roughly double the rate of those who don't. The effect is amplified in R&D, where overtime correlates with project pressure.
2. **Job role and department** — Lab Technicians and Sales Reps are structurally over-represented in attrition. This is a role-design and career-path problem, not an individual one.
3. **Monthly income (absolute and relative)** — Not just whether pay is low, but whether it is *low relative to peers at the same job level*. Employees in the bottom salary quartile for their level have disproportionately high attrition.
4. **Age and tenure** — Younger employees and those in their first year at the company are highest-risk. The model identifies this as a distinct cohort effect, not a linear relationship with age.
5. **Job satisfaction + environment satisfaction** — These interact: employees who are unsatisfied with both their role content and their work environment are far more likely to leave than those unsatisfied with just one.

**What SHRM says vs. what this data shows**

SHRM identifies compensation, career development, flexibility, and leadership as the top reasons employees leave. This dataset confirms compensation and career development but adds a nuance: *overtime combined with low pay* is the highest-risk combination — neither alone is as predictive as both together. Work-life balance ratings below 2 (on a 4-point scale) produce the most concentrated departure signal across all salary bands.

**Business implications**

These models don't just flag who might leave — they point to which interventions are most cost-effective:
- Eliminating mandatory overtime for Lab Technicians and Sales Reps would directly target the two highest-volume attrition roles
- A salary benchmarking exercise for employees at the bottom quartile of their job level would address the strongest salary signal the model finds
- A structured 90-day onboarding programme aimed at the 0–1 year tenure group would address the early-exit cohort
- Monitoring performance ratings below 3.5 combined with below-median salary is the simplest early-warning rule derivable from this data

**Model maintenance**

Like any production attrition model, this should be retrained quarterly as the workforce composition changes. Variables like `NumCompaniesWorked` and `YearsSinceLastPromotion` are sensitive to labour-market conditions that shift year to year. The Power BI dashboard tracks the key KPIs so HR can see whether the distribution of risk factors is changing over time.

---

## Tech Stack

| Tool | Use |
|---|---|
| **R / data.table / tidyverse** | Data loading, wrangling, visualisation |
| **H2O AutoML** | Automated model training and leaderboard |
| **caret / ROSE** | Manual model comparison, class balancing |
| **ggplot2 / plotly** | EDA charts |
| **Power BI** | Business-facing dashboard |

---

## References

- [IBM HR Analytics Dataset — Kaggle](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)
- [H2O AutoML documentation](https://docs.h2o.ai/)
- [SHRM — Top reasons for employee turnover](https://www.shrm.org/topics-tools/news/report-hr-pros-rank-top-reasons-turnover)
- [HireQuotient — Attrition rate benchmarks](https://www.hirequotient.com/hr-glossary/what-is-attrition-rate)
