# Global Poverty Explorer under SSP Scenarios

Predicting poverty headcount ratios under Shared Socioeconomic Pathway (SSP) scenarios using machine learning models trained on historical World Bank and IIASA data.

## Project Overview

This project builds ML models that learn the relationship between socioeconomic indicators and poverty rates from historical data (1996--2023), then projects poverty into the future (2025--2100) under three SSP scenarios:

- **SSP1** -- Sustainability (low challenges)
- **SSP4** -- Inequality (high challenges for adaptation)
- **SSP5** -- Fossil-fueled Development (high challenges for mitigation)

Three poverty thresholds are modeled: **$3/day**, **$8.30/day**, and **$10/day**.

> The $4.20/day threshold was excluded because the source data contains poverty *gap* values rather than headcount ratios.

## Features and Target

**Features (5):**
| Feature | Source |
|---|---|
| GDP per capita | World Bank (GDP) + IIASA (Population) |
| Human Development Index (HDI) | UNDP |
| Control of Corruption | World Bank Governance Indicators |
| Employment in Agriculture (%) | World Bank / ILO |
| Gini Coefficient | World Bank |

**Targets (3):** Poverty headcount ratios at $3, $8.30, $10/day thresholds.

## Models (7)

| Model | Type | Notes |
|---|---|---|
| XGBoost (CPU) | Gradient boosting | 2 000 estimators, lr 0.01, max depth 15 |
| XGBoost (GPU) | Gradient boosting | Falls back to CPU if no GPU available |
| LightGBM | Gradient boosting | |
| Random Forest | Ensemble | |
| Ridge | Linear regression | Features scaled with StandardScaler |
| MLP | Neural network | 2 hidden layers (32, 16), features scaled |
| GAM | Generalized Additive Model | Spline per feature, partial dependence plots |

## Project Structure

```
├── data/
│   ├── raw/                          # Original CSV/XLSX files
│   └── processed/                    # Cleaned outputs
│       ├── historical_panel.csv      # Merged historical data (1996–2023)
│       ├── ssp_forecast_panel.csv    # SSP feature projections (2025–2100)
│       ├── train.csv                 # 80% training split (by country)
│       ├── test.csv                  # 20% test split (by country)
│       └── predictions_ssp.csv      # Final predictions (dashboard input)
├── notebooks/
│   ├── 01_clean_historical.ipynb     # Clean & merge historical indicators
│   ├── 02_clean_ssp_forecasts.ipynb  # Clean SSP forecast features
│   ├── 03_merge_and_prepare.ipynb    # Define features/targets, train/test split
│   ├── 04_train_models.ipynb         # Train all 7 models × 3 thresholds
│   ├── 05_evaluate_and_compare.ipynb # Metrics, comparison plots
│   ├── 06_shap_analysis.ipynb        # SHAP explanations for all models
│   └── 07_predict_ssp_futures.ipynb  # Generate future poverty predictions
├── models/                           # Saved .pkl model files
├── outputs/                          # Plots and result CSVs
│   ├── shap/                         # SHAP analysis plots
│   └── trajectory_plots/             # Per-country projection plots
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

Then run the notebooks in order (01 through 07). Each notebook is self-contained: it reads its inputs from `data/` and writes its outputs back.

## Notebook Pipeline

| # | Notebook | Input | Output |
|---|---|---|---|
| 01 | Clean Historical | `data/raw/*.csv` | `historical_panel.csv` |
| 02 | Clean SSP Forecasts | `data/raw/*.csv`, `*.xlsx` | `ssp_forecast_panel.csv` |
| 03 | Merge & Prepare | `historical_panel.csv` | `train.csv`, `test.csv` |
| 04 | Train Models | `train.csv`, `test.csv` | 21 model `.pkl` files + 3 scalers |
| 05 | Evaluate & Compare | `test.csv`, models | `model_comparison.csv`, plots |
| 06 | SHAP Analysis | `train.csv`, `test.csv`, models | SHAP plots in `outputs/shap/` |
| 07 | Predict Futures | `ssp_forecast_panel.csv`, models | `predictions_ssp.csv`, trajectory plots |

## Approach A vs B

- **Approach A:** Predictions up to **2050** only. All SSP features come from published projections.
- **Approach B:** Predictions up to **2100**. Some features (Employment in Agriculture, HDI) are extrapolated beyond their source data range. Extrapolated rows are flagged in the forecast panel.

Both approaches use the same models trained on historical data. The `predictions_ssp.csv` file includes an `approach` column so the dashboard can filter accordingly.

## Data Sources

- **World Bank Open Data** -- GDP, Gini, Poverty headcount ratios, Control of Corruption, Employment in Agriculture
- **UNDP** -- Human Development Index
- **IIASA SSP Database** -- GDP and Population projections under SSP scenarios
- **SSP Extension Explorer** -- Control of Corruption, Employment in Agriculture, Gini, HDI forecasts under SSP scenarios

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
