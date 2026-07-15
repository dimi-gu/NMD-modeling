# Non Maturity Deposit Rate Modelling with an Error-Correction Model

A Python/Jupyter workflow for estimating the behaviour of **Non Maturity Deposit (NMD) customer rates** using a long-run pass-through regression and a short-run **Error-Correction Model (ECM)**.

The notebook combines Italian household deposit rates with market-rate and sovereign-spread data, performs model selection and residual diagnostics, estimates the speed of adjustment toward equilibrium, calculates mean-reversion time, and evaluates out-of-sample forecasts.

---

## Main Features

The notebook:
- Imports NMD customer rates from a Bank of Italy public database
- Imports Euribor and sovereign yields from ECB CSV files
- Converts quarterly customer-rate observations to a monthly series
- Constructs the Italian–German 10-year sovereign spread
- Adds a COVID-period dummy variable
- Performs an Augmented Dickey–Fuller stationarity test
- Estimates a long-run customer-rate model
- Applies backward regressor elimination based on:
  - statistical significance;
  - expected coefficient signs.
- Runs residual diagnostics:
  - Durbin–Watson test;
  - Q–Q plot;
  - Breusch–Pagan test.
- Reports HC1 heteroskedasticity-robust standard errors
- Estimates symmetric or asymmetric short-run pass-through
- Enforces economic restrictions on ECM coefficients
- Calculates customer-rate mean-reversion time
- Produces out-of-sample predictions for:
  - the long-run model;
  - the combined ECM specification.
- Compares forecasting errors using RMSE

---

## Input Data

Place the following input files in the same directory as the notebook:

```text

├── rate_NMD.xlsx
├── euribor1M.csv
├── ita_10y.csv
├── ger_10y.csv

```
---

## Configuration

The primary user parameter is defined in the first code cell:

```python
test_size = 18
```

| Parameter | Description |
|---|---|
| `test_size` | Number of final monthly observations reserved for out-of-sample testing |
| `test_size = 0` | Fits the model using the entire available dataset |
| `alpha` | Significance threshold for variable selection; currently 5% |

The start date (2010-06-30) and the end date (2025-12-31) are inferred from the NMD customer-rate dataset. 

---

## Usage

1. Place all four input files beside the notebook.
2. Set the desired `test_size`.
3. Run the notebook from top to bottom.
4. Review the model-selection logs, diagnostics, coefficients, mean-reversion time, and forecasting results.

---

## Notebook Workflow

### 1. Data preparation

The notebook:

- verifies whether the expected files exist;
- imports and standardises each series;
- interpolates quarterly customer rates to monthly frequency;
- merges all datasets by date;
- checks missing values;
- creates the sovereign spread and COVID dummy;
- plots the model variables.

### 2. Stationarity analysis

An Augmented Dickey–Fuller test is applied to `customer_rate`.

The saved implementation uses:

```python
adfuller(input_table["customer_rate"], regression="n")
```

This runs the test without a constant or deterministic trend.

### 3. Long-run model selection

The notebook estimates OLS models iteratively.

At each iteration it:

1. removes variables with an economically incorrect sign;
2. removes the non-constant regressor with the largest p-value above 5%;
3. stops when all remaining regressors satisfy the selected rules.

The notebook stores and displays a complete elimination log.

### 4. Long-run diagnostics

The following diagnostics are produced:

- residual time-series plot;
- Durbin–Watson statistic;
- residual Q–Q plot;
- Breusch–Pagan test;
- OLS summary with HC1 robust covariance estimates.

### 5. Short-run model selection

The short-run model is estimated only when `Euribor_1M` remains in the final long-run specification.

The notebook then selects among the asymmetric, partially asymmetric, symmetric, and theta-only models.

### 6. Forecast evaluation

The last `test_size` observations are used to evaluate:

- direct long-run-model predictions;
- ECM predictions using lagged observed customer rates.

Forecast errors and RMSE are calculated, and actual versus predicted rates are plotted.

---

## Disclaimer

This project is intended for statistical modelling, model-development, and educational purposes. It is not financial advice and should not be used for pricing, risk measurement, behavioural modelling, or regulatory decisions without independent validation, appropriate data-governance controls, and methodological review.
