# US Retail Sales — Time Series Analysis & SARIMA Forecasting

## Overview
This project analyzes 34+ years of monthly US retail sales data (1992-present) to understand seasonal spending patterns and builds a SARIMA model to forecast future sales. Retail sales is a key economic indicator, reflecting a large share of US consumer spending and closely watched to gauge the health of the economy. Data is pulled live from the FRED API rather than a static snapshot, so the analysis reflects the most recently released month at the time it's run.

## Results at a Glance
- **Final model**: SARIMA(0,1,2)(2,0,1)[12] fit on the log-transformed series, orders selected by `auto_arima`
- **Rolling-origin backtest** (24 months, refitting month by month and forecasting one month ahead each time, rather than a single train/test split): **MAPE 1.42%**, MAE ~10,185M USD, RMSE ~11,686M USD
- **6-month forecast** (Jul-Dec 2026) reproduces the historical December holiday-sales spike almost exactly in magnitude (+$79.5K forecast vs. +$64K in 2024 and +$80K in 2025), a good sign the model has learned the real seasonal pattern rather than just extrapolating trend
- **A methodological finding worth flagging**: EWMA detrending achieved stationarity on this series (p=0.011), directly contradicting the outcome in the original tutorial this project's approach was based on (which found EWMA failed on a different, energy-consumption dataset). A good reminder that transformation results don't automatically transfer between datasets — see [Known Data Issues / Limitations](#known-limitations) below.

## Key Findings

### EDA (`01_data_and_eda.ipynb`)
The NSA (not seasonally adjusted) series shows a clear repeating sawtooth pattern — December spikes, January drop-offs — that the Census Bureau's own SA (seasonally adjusted) series smooths out. A multiplicative decomposition fits better than additive: seasonal swings grow proportionally with the sales level rather than by a fixed dollar amount, consistent with the rolling standard deviation growing over the 34-year history (roughly $15-20K in the 1990s to $40-50K in recent years). An ADF test confirms the raw series is strongly non-stationary (p=0.998); the ACF decays very slowly (still ~0.65-0.7 at lag 36) while the PACF cuts off sharply after lag 1 with additional spikes at lags 12-13 — the classic signature of both a trend/unit-root problem and leftover seasonal structure.

### Transformations and stationarity (`02_transforms_stationarity.ipynb`)
Simple variance-stabilizing transforms (log, sqrt, Box-Cox) all failed to achieve stationarity on their own (p-values 0.88-0.99) — expected, since they address changing variance, not trend or seasonal structure. Moving-average detrending worked: both SMA(12) (p=0.0025, but discards the first 11 months) and EWMA(12) (p=0.011, preserves full history) succeeded. **Final approach**: log transform → first difference → seasonal (lag-12) difference, which achieved the strongest stationarity result (p≈4.9e-10) and directly matches what the ACF/PACF diagnosis predicted.

### SARIMA modeling and backtest (`03_sarima_forecasting.ipynb`)
`auto_arima` selected SARIMA(0,1,2)(2,0,1)[12] — notably with **no explicit seasonal differencing** (D=0), instead capturing the 12-month seasonal pattern through a near-unit-root seasonal AR term (`ar.S.L12` ≈ 1.08). This isn't a contradiction of the manual differencing experiment; it's a different, AIC-preferred way of handling the same underlying seasonal structure. Residual diagnostics: no leftover autocorrelation (Ljung-Box p=0.72), but non-normal residuals (Jarque-Bera p≈0.00, kurtosis ≈13), traced almost entirely to a handful of extreme points around the 2020-2021 pandemic shock rather than a broad ongoing problem.

The rolling-origin backtest — refitting on all available data each month and forecasting one month ahead, repeated across the last 24 months — reached **1.42% MAPE**, with predictions tracking the actual month-to-month zigzag (including the December/January seasonal swing) closely, not just the overall trend.

## Known Limitations
- **Residual non-normality** (driven by the 2020 pandemic shock) mainly affects the calibration of prediction intervals, not the validity of point forecasts — intervals should be treated as approximate, especially around any future shock of similar magnitude.
- **No baseline comparison yet**: the 1.42% MAPE hasn't been benchmarked against a simple baseline (e.g. seasonal naive: "this month = same month last year, adjusted for trend"), so how much the SARIMA model's complexity actually buys over a much simpler approach is still an open question.
- **Backtest window is 24 months**: this covers a relatively calm economic period; accuracy hasn't been checked across more volatile periods (e.g. around the 2020 shock itself).
- **EWMA vs. SMA stationarity result differs from the original tutorial** this project's methodology was based on — see EDA notes above. This is flagged as a reminder that per-dataset verification matters, not as an error to fix.

## Project Structure
```
├── data/
│   ├── raw/                                  # Local snapshot of the FRED pull (gitignored, or tracked if small enough)
│   └── processed/                            # (unused currently — transforms are done in-notebook)
├── notebooks/
│   ├── 01_data_and_eda.ipynb                 # Live FRED fetch, NSA vs SA comparison, decomposition, ADF/ACF/PACF
│   ├── 02_transforms_stationarity.ipynb      # Transform experiments (log/sqrt/Box-Cox/SMA/EWMA/differencing)
│   └── 03_sarima_forecasting.ipynb           # auto_arima order selection, diagnostics, rolling backtest, forecast
├── README.md
└── .gitignore
```

## Methodology Notes
- **Data collection**: live pull via the FRED API (`fredapi`), not a static file, so re-running the notebooks reflects the latest released data
- **API key handling**: loaded from a local `.env` file (via `python-dotenv`), which is gitignored and never committed
- **Stationarity**: verified independently at each transformation step via ADF testing, rather than assuming a prior project's conclusions carry over
- **Backtest design**: rolling-origin (expanding window, refit each step, one-step-ahead forecast) rather than a single fixed train/test split, to better reflect how this model would actually be used month to month

## Setup
```bash
pip install -r requirements.txt
```
Requires a free FRED API key (https://fredaccount.stlouisfed.org/apikeys). Create a `.env` file in the project root:
```
FRED_API_KEY=your_key_here
```

## Tech Stack
- Python (pandas, numpy)
- Data source: FRED API via `fredapi`
- Statistical modeling: `statsmodels` (decomposition, ADF test, SARIMAX), `pmdarima` (auto-ARIMA order selection)
- Visualization: matplotlib, seaborn

## Data Source
[FRED (Federal Reserve Economic Data)](https://fred.stlouisfed.org/), Federal Reserve Bank of St. Louis, originally sourced from the U.S. Census Bureau's Advance Monthly Sales for Retail and Food Services.

**Citation:** U.S. Census Bureau, Advance Retail Sales: Retail Trade and Food Services [RSAFSNA / RSAFS], retrieved from FRED, Federal Reserve Bank of St. Louis.
