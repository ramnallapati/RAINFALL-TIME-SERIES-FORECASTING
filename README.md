# 🌧️ Monthly Rainfall Analytics & Forecasting

An end-to-end time-series project on 38+ years of monthly rainfall: data validation, statistical inference, extreme-value modelling, leak-free forecasting with calibrated uncertainty, and extreme-month risk estimation.

> **Headline finding:** across a rolling-origin backtest and an untouched 36-month holdout, no model was statistically distinguishable from plain monthly climatology at a 12-month horizon. The project reports that honestly and focuses on calibrated uncertainty and extreme-event risk.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

---

## 📌 Problem

Water-resource, drainage and operations planning need (a) a realistic monthly rainfall outlook and (b) an honest estimate of extreme daily rainfall risk.

**Goals**
1. Forecast the next 12 months of total rainfall with prediction intervals.
2. Characterise seasonality, trends and regime shifts with proper statistical tests.
3. Quantify extreme daily rainfall via return levels.
4. Determine whether any model beats naive baselines in a statistically defensible way.

## 📂 Data

Three monthly series, **Jan 1982 – Jun 2020 (462 months)**. The layout follows the data.gov.sg monthly-rainfall datasets; units are assumed to be millimetres.

| File | Column | Meaning |
|---|---|---|
| `rainfall-monthly-total.csv` | `total_rainfall` | Total monthly rainfall (mm) |
| `rainfall-monthly-number-of-rain-days.csv` | `no_of_rainy_days` | Number of rainy days |
| `rainfall-monthly-highest-daily-total.csv` | `maximum_rainfall_in_a_day` | Highest daily total in the month (mm) |

## 🔬 Methodology

| Stage | What was done |
|---|---|
| **1. Data contract** | 7 executable checks (no gaps/duplicates/nulls, rain days ≤ days in month, max daily ≤ monthly total, etc.). The notebook fails loudly if any rule is broken. |
| **2. EDA** | Hypothesis-driven plots: seasonal profiles, year × month heat-map, distributions, frequency-vs-intensity variance decomposition. |
| **3. Statistical inference** | Mann–Kendall + Theil–Sen (trend), Pettitt (change-point), ADF + KPSS (stationarity), STL (seasonal strength), ACF/PACF, Ljung–Box. Multiple-testing caveat applied. |
| **4. Extreme values** | GEV fit to annual maxima of daily rainfall, QQ-plot, KS test, return levels with bootstrap 95% CIs. |
| **5. Feature engineering** | Calendar harmonics and lag features with **lag ≥ 12 only**, so no feature uses information unavailable at forecast time. Includes an automated leakage unit-test. |
| **6. Modelling** | Seasonal-naive, climatology, ETS (Holt-Winters), DHR-ARIMA (Fourier terms + ARMA errors), gradient boosting (gamma loss), and an equal-weight ensemble. |
| **7. Validation** | Expanding-window **rolling-origin backtest** (8 CV folds, Jul 2009 – Jul 2016) plus an **untouched 36-month holdout** (Jul 2017 – Jun 2020). Metrics: MAE, RMSE, MASE, bias, RMSLE. Paired Wilcoxon tests between models. |
| **8. Uncertainty** | Split-conformal 90% prediction intervals calibrated on CV residuals only; coverage checked on the holdout. |
| **9. Extreme-month risk** | Beta-posterior risk by calendar month and a logistic model scored with Brier skill and AUC. |
| **10. Production forecast** | Refit on all data and forecast Jul 2020 – Jun 2021 with intervals. |

## 📊 Key results

| Finding | Result |
|---|---|
| Seasonality | Real but moderate (STL seasonal strength 0.35). Wettest months on average: Dec, Nov, Jan; wettest/driest month ratio ≈ 3×. |
| Frequency vs intensity | Rain-day frequency explains ~43% and intensity ~57% of variance in log monthly rainfall. |
| Annual total trend | No significant trend (Mann–Kendall p = 0.48). |
| Rainy days / intensity | Rainy days per year trending up (p ≈ 0.045); rain per rainy day trending down (p ≈ 0.015, Pettitt shift around 2007). Only the intensity result is near the Bonferroni threshold (0.0125). |
| Extremes | GEV tail index ξ ≈ −0.14. 10-year daily return level ≈ 187 mm (95% CI 160–207); 50-year ≈ 232 mm (CI 186–282). Record: 216 mm (Jan 2011). |
| Forecast accuracy (holdout MAE) | Ensemble 73.4 mm · Climatology 74.7 mm · Seasonal-naive 82.2 mm |
| Significance | Ensemble beats seasonal-naive (p ≈ 0.01) but is **not** distinguishable from climatology (p ≈ 0.34). |
| Interval calibration | 90% intervals reached 89% holdout coverage, but are wide (mean width ≈ 2× mean monthly rainfall). |
| Bias | All models over-forecast the holdout (≈ +40 mm): 2017–2020 was drier than the earlier record. |
| Extreme-month classifier | Brier skill ≈ −6%, AUC ≈ 0.73 on 120 months with only 4 events. Little beyond calendar seasonality can be claimed. |

## 🗂️ Repository structure

```
.
├── Rainfall_Analytics_and_Forecasting.ipynb   # main notebook (all analysis)
├── README.md
├── requirements.txt
├── data/
│   ├── rainfall-monthly-total.csv
│   ├── rainfall-monthly-number-of-rain-days.csv
│   └── rainfall-monthly-highest-daily-total.csv
└── outputs/                                   # created when the notebook runs
    ├── forecast_next_12_months.csv
    ├── backtest_predictions.csv
    ├── backtest_metrics.csv
    ├── gev_return_levels.csv
    └── model_card.json
```

## 🚀 How to run

```bash
git clone https://github.com/ramnallapati/RAINFALL-TIME-SERIES-FORECASTING.git
cd RAINFALL-TIME-SERIES-FORECASTING
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook Rainfall_Analytics_and_Forecasting.ipynb
```

Place the three CSV files in `./data` (or next to the notebook). Then run **Kernel → Restart & Run All**. Runtime is a few minutes, mostly model refits in the backtest.

**`requirements.txt`**
```
numpy
pandas
scipy
scikit-learn
statsmodels
matplotlib
seaborn
jupyter
```

## ⚠️ Limitations

1. Single aggregated series with no exogenous drivers (e.g. ENSO or Indian Ocean Dipole indices). These are the most credible route to extra forecasting skill.
2. Monthly resolution hides storm structure; flood risk depends on sub-daily intensity.
3. 38 annual maxima give wide GEV intervals; long return periods (50–100 years) are order-of-magnitude estimates only.
4. Station and spatial coverage of the source data is unknown.
5. The extreme-month evaluation is data-starved (few events), so Brier/AUC carry wide uncertainty.

## 🔭 Possible extensions

- Add lagged climate-mode indices as predictors
- Direct quantile gradient boosting and CRPS scoring
- Non-stationary GEV with covariates
- Bayesian structural time-series with partial pooling across months
- Automated retraining and monitoring (rolling MASE, interval coverage)

## 🛠️ Tech stack

Python · pandas · NumPy · SciPy · statsmodels · scikit-learn · matplotlib · seaborn · Jupyter

## 👤 Author

**Nallapati Rama Rao**: AI/ML Engineer
[LinkedIn](#https://www.linkedin.com/in/rama-rao-nallapati-42a659313/) · [GitHub](#https://github.com/ramnallapati)

