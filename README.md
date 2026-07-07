# Home Depot (HD): Stylised Facts, CAPM/APT, and ARMA-GARCH Forecasting

Personal project applying time-series econometrics and asset-pricing models to
Home Depot (NYSE: HD) daily and hourly returns, including a GARCH(1,1)
volatility-modelling extension.

## What this project does

1. **Stylised facts** — computes daily (2y) and hourly (60d) log-returns and checks
   them against well-known stylised facts of financial returns: near-zero mean,
   volatility clustering, and fat-tailed, non-normal distributions (skewness,
   excess kurtosis).
2. **Data verification** — cross-checks S&P 500 returns and T-bill rates against
   the Fama-French data library before using either as inputs downstream.
3. **CAPM** — estimates HD's market beta and builds a replication portfolio to
   compare risk-adjusted performance (Sharpe ratio) against the stock itself.
4. **APT (Fama-French five factors + momentum + oil)** — extends CAPM to a
   multi-factor model, tests whether momentum adds explanatory power, and adds
   an oil-price factor to capture supply-chain/input-cost sensitivity. Checks
   for multicollinearity via VIF.
5. **ARMA modelling** — grid-searches low-order ARMA(p,q) specifications by
   AIC/BIC, runs Ljung-Box and BDS residual diagnostics, produces a long-run
   dynamic forecast, and evaluates out-of-sample RMSE against a naive
   white-noise benchmark.
6. **GARCH volatility modelling** — the ARMA residual diagnostics in step 5
   show persistent autocorrelation in squared residuals and a BDS test that
   strongly rejects i.i.d. residuals, both signs that conditional variance
   needs its own model. This section runs a formal Engle ARCH-LM test, fits a
   GARCH(1,1) on the ARMA residuals, and compares constant-width ARMA-only
   forecast intervals against the time-varying ARMA+GARCH intervals.

## Key idea

Return **levels** for a large, mature stock like HD are close to unpredictable
(ARMA barely beats white noise on point forecasts) — but return **volatility**
is highly predictable in the short run. The GARCH extension makes that second
point explicit and quantifiable, which matters for risk management (VaR,
options pricing) even when point forecasting is close to hopeless.

## Results

| Model | Result |
|---|---|
| CAPM | β = 1.089, α ≈ 0.0006 |
| Fama-French 5-factor | adj. R² = 0.294 |
| FF5 + Momentum + Oil | adj. R² = 0.456 |
| Multicollinearity (VIF) | all factors 1.03–1.63 (well below 5) |
| Best ARMA by AIC | ARMA(2,1) |
| ARMA out-of-sample RMSE | ≈ white-noise benchmark (0.0168 vs 0.0168) |
| Engle ARCH-LM test | statistic = 674.8, p ≈ 1.6×10⁻¹³⁸ — strong ARCH effects |
| GARCH(1,1) | α = 0.066, β = 0.930 (highly persistent volatility) |
| Forecast interval half-width | ARMA-only: 0.0445 (constant) · ARMA+GARCH: 0.0272–0.0381 (time-varying) |

The ARMA+GARCH intervals are consistently tighter than the ARMA-only interval
and move with market conditions, which is the practical payoff of modelling
conditional variance separately from the conditional mean.

## Repo structure

```
notebooks/
  hd_financial_econometrics.ipynb   # full analysis, runnable end-to-end
assets/                              # figures exported from the notebook
requirements.txt
```

## Running it

```bash
pip install -r requirements.txt
jupyter notebook notebooks/hd_financial_econometrics.ipynb
```

Data is pulled live via `yfinance` (Yahoo Finance) and the Kenneth French Data
Library, so an internet connection is required. No API keys needed.

## Data sources

- Prices: Yahoo Finance (`yfinance`) — HD, `^GSPC` (S&P 500), `^IRX` (13-week
  T-bill), `CL=F` (WTI crude oil)
- Factors: [Kenneth French Data Library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html)
  — Fama-French five factors (daily) + momentum (daily)

## Author

Yurui Qiu — Bachelor of Data Science and Decisions, UNSW Sydney
[GitHub](https://github.com/DDDaaa11) · [Portfolio](https://dddaaa11.github.io/retail-turnover-forecasting)
