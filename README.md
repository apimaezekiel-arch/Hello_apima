# SPY Volatility Forecasting
A financial engineering project comparing GARCH, EWMA,
and rolling historical volatility forecasts for SPY.
## Research question
Can GARCH forecast future volatility more accurately
than simpler forecasting models?
## Models
- GARCH(1,1)
- EWMA
- 20-day rolling historical volatility
## Evaluation
Forecasts are compared with subsequently realized
volatility using MAE and RMSE.
## Project status
Code prepared. Evaluation on real SPY data is pending.
## Author
Ezekiel Apima Onchong’a
MSc Financial Engineering student, WorldQuant University
# Setup

Requires Python 3.10+.

```bash
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python -m src.forecast --start 2018-01-01 --end 2026-01-01
pytest -q
```

For your own licensed or publicly shareable daily SPY CSV:

```bash
python -m src.forecast --csv path/to/spy.csv --start 2018-01-01 --end 2026-01-01
```

CSV needs `Date` and `Close` or `Adj Close`. Default download uses yfinance and adjusted closing prices. End date is exclusive. Closing prices must be consistent within a single series. Output in `results/`: dated `forecasts.csv`, `metrics.csv`, and `comparison.png`.

## Method

1. Compute daily log returns from adjusted prices.
2. At each forecast origin, fit models using returns available through that date only (minimum 750 daily returns by default).
3. Forecast the next 20 trading days. The rolling and EWMA models hold their daily variance forecast constant across the horizon; GARCH projects its daily variance separately for each step. Average projected daily variances, then take the square root and annualize by 252 trading days.
4. Compare to the root mean squared return over the subsequent 20 days, annualized by 252. This is realized quadratic variation, without subtracting the future sample mean.
5. Move 20 trading days forward and repeat. These default test windows do not overlap. The training window expands; GARCH fits using the latest 750 returns to control run time. No future returns enter a forecast.

Volatility is reported as a decimal (0.20 = 20% annualized). The GARCH model uses zero conditional mean and normal innovations; returns are multiplied by 100 for fitting and converted back afterward. These choices are explicit research assumptions, not optimized hyperparameters.

## Interpret results

Open `results/metrics.csv` and identify the lowest out-of-sample MAE and RMSE. Inspect `comparison.png` for stress periods. Report sample dates, number of forecast origins, horizon, and any missing days. Do not claim one model is superior before running the experiment on actual data. Consider a separate robustness run at 5 days using `--horizon 5 --step 5`.

## Capstone extension

Use each origin's GARCH volatility in Black–Scholes and compare option-price errors against a calibrated Heston model. This requires historical option bid/ask quotes with timestamp, strike, expiry, underlying price, dividends, and rate. Current option chains cannot reconstruct historical quotes; respect data-provider licensing. For American-style SPY options, account for early exercise and dividends when interpreting Black–Scholes/Heston comparisons.

## Repository contents

- `src/forecast.py`: ingestion, modeling, backtest, scoring and figure.
- `tests/test_forecast.py`: window alignment and basic score checks.
- `requirements.txt`: dependencies.
- `.gitignore`: generated and local files excluded from Git.
