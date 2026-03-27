# Global Poverty Explorer under SSP Scenarios

Predicting poverty headcount ratios under Shared Socioeconomic Pathway (SSP) scenarios using machine learning models trained on historical World Bank and IIASA data. The repository now also includes an interactive Streamlit dashboard for exploring the final SSP projections.

## Project Overview

This project builds ML models that learn the relationship between socioeconomic indicators and poverty rates from historical data (1996--2023), then projects poverty into the future (2025--2100) under SSP scenarios.

The final interactive dashboard lets users:
- explore predicted poverty headcount ratios by **scenario**, **year**, **poverty line**, and **region**
- compare distributions across scenarios
- inspect continent-level trajectories over time
- generate SHAP-based explainability views from the final XGBoost models

Three poverty thresholds are modeled:
- **$3/day**
- **$8.30/day**
- **$10/day**

> The $4.20/day threshold was excluded because the source data contains poverty *gap* values rather than headcount ratios.

## Features and Targets

**Features (5):**

| Feature | Source |
|---|---|
| GDP per capita | World Bank (GDP) + IIASA (Population) |
| Human Development Index (HDI) | UNDP |
| Control of Corruption | World Bank Governance Indicators |
| Employment in Agriculture (%) | World Bank / ILO |
| Gini Coefficient | World Bank |

**Targets (3):**
- Poverty headcount ratio at **$3/day**
- Poverty headcount ratio at **$8.30/day**
- Poverty headcount ratio at **$10/day**

## Models

| Model | Type | Notes |
|---|---|---|
| XGBoost (CPU) | Gradient boosting | Final dashboard explainability models |
| XGBoost (GPU) | Gradient boosting | Falls back to CPU if no GPU available |
| LightGBM | Gradient boosting | |
| Random Forest | Ensemble | |
| Ridge | Linear regression | Features scaled with StandardScaler |
| MLP | Neural network | 2 hidden layers (32, 16), features scaled |
| GAM | Generalized Additive Model | Spline per feature, partial dependence plots |

## Project Structure

```text
├── data/
│   ├── raw/                           # Original CSV/XLSX files
│   └── processed/                     # Cleaned outputs used across notebooks/app
│       ├── historical_panel.csv       # Merged historical data (1996–2023)
│       ├── ssp_forecast_panel.csv     # SSP feature projections (2025–2100)
│       ├── train.csv                  # 80% training split (by country)
│       ├── test.csv                   # 20% test split (by country)
│       └── predictions_ssp.csv        # Final dashboard input
├── notebooks/
│   ├── 01_clean_historical.ipynb      # Clean & merge historical indicators
│   ├── 02_clean_ssp_forecasts.ipynb   # Clean SSP forecast features
│   ├── 03_merge_and_prepare.ipynb     # Define features/targets, train/test split
│   ├── 04_train_models.ipynb          # Train all models
│   ├── 05_evaluate_and_compare.ipynb  # Metrics and comparison plots
│   ├── 06_shap_analysis.ipynb         # SHAP explanations for model comparison
│   ├── 07_predict_ssp_futures.ipynb   # Generate future poverty predictions
│   └── 08_dashboard_app.ipynb         # Documents the Streamlit dashboard code
├── models/                            # Saved .pkl model files
│   ├── XGBoost_CPU_poverty_3.pkl
│   ├── XGBoost_CPU_poverty_8_30.pkl
│   └── XGBoost_CPU_poverty_10.pkl
├── outputs/                           # Plots and result CSVs
│   ├── shap/                          # SHAP analysis plots
│   └── trajectory_plots/              # Per-country projection plots
├── dashboard_app.py                   # Runnable Streamlit dashboard
├── requirements.txt
├── LICENSE
└── README.md
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Notebook Pipeline

Run the notebooks in order:

| # | Notebook | Input | Output |
|---|---|---|---|
| 01 | Clean Historical | `data/raw/*.csv` | `historical_panel.csv` |
| 02 | Clean SSP Forecasts | `data/raw/*.csv`, `*.xlsx` | `ssp_forecast_panel.csv` |
| 03 | Merge & Prepare | `historical_panel.csv` | `train.csv`, `test.csv` |
| 04 | Train Models | `train.csv`, `test.csv` | model `.pkl` files + scalers |
| 05 | Evaluate & Compare | `test.csv`, models | `model_comparison.csv`, plots |
| 06 | SHAP Analysis | `train.csv`, `test.csv`, models | SHAP plots in `outputs/shap/` |
| 07 | Predict Futures | `ssp_forecast_panel.csv`, models | `predictions_ssp.csv`, trajectory plots |
| 08 | Dashboard App | `predictions_ssp.csv`, XGBoost CPU models | documented dashboard code |

## Dashboard

The dashboard is built with Streamlit and reads the final prediction table from:

```text
data/processed/predictions_ssp.csv
```

For SHAP-based explainability, it loads the final XGBoost CPU models from:

```text
models/XGBoost_CPU_poverty_3.pkl
models/XGBoost_CPU_poverty_8_30.pkl
models/XGBoost_CPU_poverty_10.pkl
```

### Run the dashboard

From the project root:

```bash
streamlit run dashboard_app.py
```

### Dashboard views

- **Risk Map**: country-level poverty headcount ratio by SSP scenario and year
- **Scenario Comparison**: cross-scenario comparison for the selected year
- **Regional Trends**: continent-level trajectories over time
- **Explainability**: SHAP-compatible global and country-level explanations from the final XGBoost models

## Approach A vs B

- **Approach A:** Predictions up to **2050** only. All SSP features come from published projections.
- **Approach B:** Predictions up to **2100**. Some features are extrapolated beyond their original source range.

Both approaches use the same models trained on historical data. If the prediction file contains an `approach` column, the dashboard keeps **Approach B** rows for the interactive view.

## Data Sources

- **World Bank Open Data** -- GDP, Gini, Poverty headcount ratios, Control of Corruption, Employment in Agriculture
- **UNDP** -- Human Development Index
- **IIASA SSP Database** -- GDP and Population projections under SSP scenarios
- **SSP Extension Explorer** -- Control of Corruption, Employment in Agriculture, Gini, HDI forecasts under SSP scenarios

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
