# US Retail Sales — Time Series Analysis & SARIMA Forecasting

## Overview
This project analyzes 30+ years of monthly US retail sales data to understand seasonal patterns and consumer spending trends, and builds a SARIMA model to forecast future retail sales. Retail sales is a key economic indicator — it reflects a large share of US consumer spending and is closely watched by economists and analysts to gauge the health of the economy.

## Data Source
[FRED (Federal Reserve Economic Data)](https://fred.stlouisfed.org/), Federal Reserve Bank of St. Louis, sourced originally from the U.S. Census Bureau.

- **`RSAFSNA`** — Advance Retail Sales: Retail Trade and Food Services, **Not Seasonally Adjusted** (millions of USD, monthly)
- **`RSAFS`** — same series, **Seasonally Adjusted** (used for comparison against our own seasonal adjustment work)

Both series run from January 1992 to the most recently released month, and are updated monthly via the FRED API.

**Citation:** U.S. Census Bureau, Advance Retail Sales: Retail Trade and Food Services [RSAFSNA/RSAFS], retrieved from FRED, Federal Reserve Bank of St. Louis.

## Project Type
- Time series EDA (decomposition, stationarity testing)
- Statistical transformation experiments (log transform, differencing)
- SARIMA forecasting with rolling backtest evaluation

## Approach
1. **Data collection**: pull live data via the FRED API (`fredapi`) rather than a static snapshot, so the analysis always reflects the most current release
2. **EDA**: visualize NSA vs. SA series, seasonal decomposition (multiplicative vs. additive), rolling statistics, ACF/PACF
3. **Stationarity**: ADF testing on the raw series, followed by transformation experiments (log transform, first/seasonal differencing) until stationarity is achieved
4. **Modeling**: SARIMA model fit on the transformed series, with a rolling-origin backtest (forecast one month ahead, compare to the actual value once released, repeat) rather than a single train/test split — this better reflects how the model would actually be used
5. **Evaluation**: forecast accuracy (MAE/RMSE/MAPE) plus qualitative check of whether prediction intervals are well-calibrated

## Project Structure
```
├── data/
│   ├── raw/                       # Local snapshot of FRED pull (gitignored)
│   └── processed/                 # Transformed/differenced series (gitignored)
├── notebooks/
│   ├── 01_data_and_eda.ipynb      # Data fetch, decomposition, stationarity diagnostics
│   ├── 02_transforms_stationarity.ipynb  # (planned) transformation experiments
│   └── 03_sarima_forecasting.ipynb        # (planned) SARIMA modeling and backtest
├── src/
├── README.md
├── requirements.txt
└── .gitignore
```

## Setup

```bash
pip install -r requirements.txt
```

This project requires a free FRED API key:
1. Register at https://fredaccount.stlouisfed.org/apikeys
2. Set it as an environment variable (never hardcode it in a notebook):
   ```bash
   export FRED_API_KEY="your_key_here"
   ```

## Tech Stack
- Python (pandas, numpy)
- Data source: FRED API via `fredapi`
- Statistical modeling: `statsmodels` (decomposition, ADF test, SARIMA), `pmdarima` (auto-ARIMA order selection)
- Visualization: matplotlib, seaborn

## Status
🚧 In progress — data collection and EDA phase
