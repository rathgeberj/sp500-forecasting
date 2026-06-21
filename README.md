# S&P 500 Forecasting with Macroeconomic Indicators

Forecasting next-day S&P 500 log returns from macroeconomic and technical features,
comparing classical statistical, tree-based, and deep-learning models.

The full pipeline — data cleaning, exploratory analysis, stationarity testing, model
training, evaluation, and feature engineering — lives in a single Jupyter/Colab
notebook: [`2_0_Data_Notebook.ipynb`](2_0_Data_Notebook.ipynb).
A slide deck summarizing the work is in
[`SP500_ML_Full_Presentation.pptx.pdf`](SP500_ML_Full_Presentation.pptx.pdf).

## Data

Roughly 10 years of daily data (2016-04-25 → 2026-04-24, 2,515 trading days). Each
macro series is aligned to the S&P 500 trading-day calendar and forward-filled to a
daily frequency.

| File | Series | Native frequency |
|------|--------|------------------|
| `^spx_d.xlsx` | S&P 500 OHLCV | Daily |
| `fedfundrate.xlsx` | Federal funds rate | Daily |
| `GDP.xlsx` | GDP (billions USD) | Quarterly |
| `inflation.xlsx` | Core sticky inflation (CPI) | Monthly |
| `unemploy.xlsx` | Unemployment rate | Monthly |

Outliers (e.g., the COVID-19 unemployment spike to 14.8%, the 2022–2023 rate-hike
cycle) are detected but **kept** — they represent real economic events that carry the
most signal.

## Pipeline

1. **Data cleaning & processing** — load all five sources, de-duplicate, align to
   trading days, forward/backward-fill macro gaps, time-interpolate price gaps, and
   merge into one master DataFrame.
2. **Exploratory data analysis** — indicator time series, log returns with 30-day
   rolling annualized volatility, a correlation heatmap, and 30/90-day moving-average
   crossovers (golden/death crosses).
3. **Stationarity** — Augmented Dickey-Fuller (ADF) tests; non-stationary series are
   first-differenced until stationary. Log returns are already stationary.
4. **Modeling** — predict next-day log return from lagged macro features, using a
   time-ordered 80/20 split (no shuffling, no look-ahead leakage).
5. **Evaluation** — MAE, RMSE, and MAPE, plus actual-vs-predicted and residual plots.
6. **Feature engineering** — technical indicators (RSI-14, MACD, Bollinger %B),
   multi-lag features (lags 1/2/3/5/10), and interaction terms
   (fed × inflation, GDP × unemployment), expanding to 32 features.
7. **XGBoost** on the expanded feature set, with an updated model comparison.

## Models

| Model | Notes |
|-------|-------|
| Linear Regression | Baseline floor |
| Random Forest | Non-linear regime shifts |
| VAR | Joint dynamics / feedback across series |
| LSTM | 30-day sequence memory |
| XGBoost | Gradient boosting on the expanded feature set |

## Results

Test period: **2024-04 → 2026-04** (~500 days). Best model by RMSE: **XGBoost**.

| Model | MAE | RMSE | MAPE (%) |
|-------|-----|------|----------|
| Linear Regression | 0.006775 | 0.010242 | 169.24 |
| Random Forest | 0.006770 | 0.010237 | 171.07 |
| VAR | 0.006859 | 0.010272 | 243.99 |
| LSTM | 0.006743 | 0.010247 | 191.94 |
| **XGBoost** | **0.004569** | **0.007231** | 526.14 |

The four base models cluster tightly just above the linear baseline — lagged macro
data alone has weak day-ahead predictive power. XGBoost on the engineered feature set
(technical indicators + multi-lag + interactions) is the clear RMSE winner; its high
MAPE reflects the usual instability of percentage error on near-zero return days.

## Running it

The notebook was written for **Google Colab** and begins with a `google.colab.files`
upload cell. To run locally, replace that cell with direct reads of the `.xlsx` files
(the loaders already reference the local filenames) and install the dependencies:

```bash
pip install pandas numpy scipy matplotlib seaborn statsmodels scikit-learn xgboost tensorflow openpyxl
```

Then open `2_0_Data_Notebook.ipynb` and run the cells top to bottom.

## Repository layout

```
2_0_Data_Notebook.ipynb           Full analysis & modeling notebook
SP500_ML_Full_Presentation.pptx.pdf   Summary slide deck
^spx_d.xlsx                        S&P 500 daily OHLCV
fedfundrate.xlsx                   Federal funds rate
GDP.xlsx                           GDP
inflation.xlsx                     Inflation (CPI)
unemploy.xlsx                      Unemployment rate
```
