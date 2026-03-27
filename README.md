# IMO Education Analysis – Predictive Modeling

**Individual project** | UMass Amherst, Fall 2024  
**Author:** Saniya Bekova

---

## Overview

This project investigates how a country's education system impacts its performance in the **International Mathematical Olympiad (IMO)**. Specifically, it explores the relationship between government spending on education, literacy rates, tertiary enrollment, and a country's success in the IMO (measured as average score per contestant).

Three regression models are compared: **Linear Regression**, **Gradient Boosting (LightGBM)**, and **Generalized Additive Models (GAM)**.

---

## Research Question

> Does a country's investment in education predict its performance in the International Mathematical Olympiad?

---

## Data Sources

| Dataset | Source | Years |
|---------|--------|-------|
| IMO performance data | [TidyTuesday GitHub](https://github.com/rfordatascience/tidytuesday/blob/master/data/2024/2024-09-24/readme.md) | 1959–2024 |
| Educational indicators | [UNESCO UIS](https://sdg4-data.uis.unesco.org/) | 2013–2024 |
| Additional education stats | [World Bank DataBank](https://databank.worldbank.org/source/education-statistics-%5e-all-indicators) | 2009–2019 |
| Literacy rates | World Bank Education Statistics | 2009–2019 |

**Final dataset:** 1,142 observations × 38 columns (filtered to 2009–2019)

---

## Features Used

| Feature | Description |
|---------|-------------|
| `team_size_all` / `team_size_male` / `team_size_female` | Team composition |
| `Value_gross_enr_ratio_for_tertirary_edu` | Gross enrollment ratio for tertiary education |
| `Value_gov_expen_as_perc_of_GDP` | Government expenditure as % of GDP |
| `Value_literacy_rate` | Youth literacy rate (15–24 years) |
| `UIS.X.USCONST.FSGOV` | Gov. expenditure on education (constant US$ millions) |
| `UIS.X.US.FSGOV` | Gov. expenditure on education (US$ millions) |
| `NY.GDP.PCAP.CD` | GDP per capita (current US$) |
| `SE.XPD.TOTL.GB.ZS` | Education expenditure as % of total gov. expenditure |
| `SE.XPD.CUR.TOTL.ZS` | Current expenditure as % of total public institution expenditure |
| `OECD.TSAL.1.E10` | Teacher salaries (USD, primary, 10 years experience) |
| `SL.TLF.ADVN.ZS` | Labor force with advanced education (%) |
| `IT.NET.USER.P2` | Internet users per 100 people |
| `SE.TER.GRAD.SC.ZS` | Graduates in Natural Sciences, Math & Statistics (%) |

**Engineered features:**
- `Medal_Efficiency` — medals won / team size
- `Gov_Investment_Per_Medal` — government expenditure / medals won
- `Lit_Performance_Ratio` — literacy rate / average IMO score

**Target variable:** `average_score_per_contestant` — total team score divided by team size

---

## Methodology

### Data Preprocessing
- Missing literacy rate values imputed using regional averages, then income group averages, then mean
- Near-zero variance predictors removed (`step_nzv`)
- Numeric predictors normalized (`step_normalize`)
- Train/test split: **75/25**, stratified on `Value_gov_expen_as_perc_of_GPP`

### Models

#### 1. Linear Regression
Baseline model. Assumes linear relationships between predictors and target.

#### 2. Gradient Boosting (LightGBM)
Hyperparameter tuning via Bayesian optimization with 6-fold cross-validation:
- `trees`: 500–3000
- `tree_depth`: 1–5  
- `learn_rate`: 0.01, 0.05, 0.1

#### 3. Generalized Additive Model (GAM)
Uses cubic regression splines (`bs = "cr"`, `k = 10`) to model nonlinear relationships between each predictor and the outcome.

---

## Results

| Model | Train RMSE | Test RMSE | Test R² |
|-------|-----------|-----------|---------|
| Linear Regression | 6.38 | 6.46 | 0.398 |
| **Gradient Boosting** | **3.13** | **4.71** | **0.684** |
| GAM | ~0.00 | 1.40 | 0.772 |

> **Best model: Gradient Boosting** — best balance of accuracy and generalization. GAM achieves highest test R² (0.772) but severely overfits (train R² = 1.0).

---

## Key Findings

- **Government expenditure on education** (US$ millions) shows a positive correlation with average IMO score — countries that spend more tend to perform better
- **Literacy rate alone** does not strongly predict IMO performance (similar score distributions across literacy rate ranges)
- **Tertiary enrollment ratio** is a significant positive predictor (p < 0.001 in linear regression)
- **Team size male** is the strongest individual predictor in linear regression (β = 2.62, p < 0.001)
- None of the models perfectly captured the complexity of the relationship — IMO performance is influenced by many factors beyond education spending

---

## Visualizations

- Scatter plot: Government expenditure vs. average IMO score (with colorblind-friendly CVD grid)
- Line plot: Medal counts of top 3 countries (2009–2019)
- Density plot: IMO score distributions by literacy rate range
- Boxplot: IMO scores by female team presence
- GAM partial dependency plots for all predictors

---

## Tech Stack

```r
library(tidymodels)   # modeling framework
library(bonsai)       # LightGBM integration
library(themis)       # class imbalance handling
library(mgcv)         # GAM
library(ggplot2)      # visualization
library(ggrepel)      # label placement
library(colorblindr)  # CVD-friendly color grids
library(colorspace)   # color utilities
library(lubridate)    # date handling
library(readxl)       # Excel file reading
```

---

## Repository Structure

```
imo-education-analysis/
├── README.md
├── .gitignore
├── proj_report_f24_sbekova.qmd    # Full Quarto report with all code and analysis
├── proj_report_f24_sbekova.pdf    # Rendered PDF report
├── Upload_Other_Features.qmd      # Additional feature engineering (World Bank indicators)
└── data/                          # Datasets (UNESCO, World Bank, IMO, merged)
```

---

## Ethical Considerations

The model may reflect systemic inequalities — countries with higher government spending or GDP naturally have advantages in both education quality and IMO performance. Care should be taken not to use such models to justify further resource concentration in already advantaged countries.
