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

Place the following four files in the same directory as the notebook:

```text
.
├── NMD_Rate_ECM.ipynb
├── rate_NMD.xlsx
├── euribor1M.csv
├── ita_10y.csv
├── ger_10y.csv
└── README.md
```

### 1. `rate_NMD.xlsx`

Source: Bank of Italy statistical database.

The notebook reads the sheet:

```text
Report
```

Expected structure:

| First column | Producer households | Other optional columns |
|---|---:|---:|
| Date | Customer rate | ... |

Processing steps:

1. the first column is renamed `DATE`;
2. `DATE` is parsed using day-first format;
3. empty columns are removed;
4. `Producer households` is renamed `customer_rate`;
5. quarterly observations are reindexed to month-end;
6. missing monthly rates are filled using linear interpolation.

### 2. `euribor1M.csv`

One-month Euribor data from the ECB Data Portal.

Expected structure:

| DATE | Value column |
|---|---:|
| 2010-01-31 | 0.42 |
| 2010-02-28 | 0.39 |

ECB series used in the notebook:

```text
FM.M.U2.EUR.RT.MM.EURIBOR1MD_.HSTA
```

### 3. `ita_10y.csv`

Italian 10-year government bond yield from the ECB Data Portal.

ECB series:

```text
IRS.M.IT.L.L40.CI.0000.EUR.N.Z
```

### 4. `ger_10y.csv`

German 10-year government bond yield from the ECB Data Portal.

ECB series:

```text
IRS.M.DE.L.L40.CI.0000.EUR.N.Z
```

### CSV requirements

Each market-data CSV must contain:

- a `DATE` column;
- one value column.

Columns whose names contain `TIME PERIOD` or `TIME_PERIOD` are removed automatically. The remaining value column is renamed according to the input dictionary.

---

## Constructed Variables

### Sovereign spread

The Italian–German sovereign spread is calculated as:

\[
\mathrm{Spread}_t
=
\mathrm{Italy10Y}_t-\mathrm{Germany10Y}_t
\]

### COVID dummy

The notebook sets:

```python
dCovid = 1
```

for observations from **28 February 2020 through 31 December 2020**, and zero otherwise.

### Units

The source market rates are assumed to be expressed in percentage points. Before estimation, the notebook divides:

- `Euribor_1M`;
- `customer_rate`;
- `Spread`

by 100, so model coefficients operate on decimal rates.

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

The start and end dates are inferred from the NMD customer-rate dataset.

---

## Installation

Install the required packages with:

```bash
pip install numpy pandas matplotlib statsmodels openpyxl jupyter
```

Start Jupyter with:

```bash
jupyter notebook
```

---

## Usage

1. Download the market-rate CSV files from the ECB Data Portal.
2. Export the NMD customer-rate workbook from the Bank of Italy database.
3. Rename the files exactly as required by the notebook.
4. Place all four input files beside the notebook.
5. Set the desired `test_size`.
6. Run the notebook from top to bottom.
7. Review the model-selection logs, diagnostics, coefficients, mean-reversion time, and forecasting results.

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
- one-step ECM predictions using lagged observed customer rates.

Forecast errors and RMSE are calculated, and actual versus predicted rates are plotted.

---

## Outputs

The notebook generates:

1. merged monthly modelling dataset;
2. missing-value report;
3. time-series chart of Euribor, customer rates, and sovereign spread;
4. ADF stationarity-test result;
5. long-run variable-elimination log;
6. long-run OLS summary;
7. residual time-series plot;
8. Durbin–Watson diagnostic;
9. residual Q–Q plot;
10. Breusch–Pagan test;
11. HC1 robust regression summary;
12. short-run variable-elimination log;
13. short-run OLS summary;
14. estimated mean-reversion time;
15. long-run out-of-sample forecast table;
16. long-run forecast RMSE;
17. ECM out-of-sample forecast table;
18. ECM forecast RMSE;
19. chart comparing actual rates with both forecast approaches.

---

## Important Methodological Notes

### Cointegration should be tested explicitly

An ECM is appropriate when the level variables are non-stationary but share a stable cointegrating relationship.

The notebook currently tests only the customer-rate series and does not explicitly test:

- every level variable for integration order;
- the first differences;
- the long-run residuals for stationarity;
- cointegration using an Engle–Granger or Johansen procedure.

Without evidence of cointegration, a levels regression may be spurious and the ECM interpretation may not be valid.

### Interpolated observations are modelled as if observed

Quarterly customer rates are converted to monthly frequency with linear interpolation. The resulting monthly values are estimates rather than independently observed rates.

This affects:

- sample size;
- residual serial correlation;
- estimated significance levels;
- apparent forecasting accuracy.

Where possible, model the original reporting frequency or explicitly account for interpolation.

### HC1 does not correct the residual process

HC1 changes the estimated covariance matrix and standard errors. It does not:

- alter the OLS coefficients;
- alter the residuals;
- remove heteroskedasticity;
- remove autocorrelation.

Therefore, re-running the Breusch–Pagan test on `get_robustcov_results(...).resid` will return the same residual-based result. Strong residual autocorrelation may require a dynamic specification or a covariance estimator that is robust to serial correlation, such as HAC/Newey–West.

### Durbin–Watson bounds are hard-coded

The decision thresholds in the notebook are manually entered for a specific sample size and number of regressors:

```python
if DW < 1.738:
    ...
elif DW > 1.799:
    ...
```

These bounds should be updated whenever the estimation sample or model specification changes.

### The short-run regression has no intercept

The ECM short-run model is fitted without a constant. Its reported \(R^2\) is therefore an uncentred \(R^2\), which is not directly comparable with the standard centred \(R^2\) from the long-run regression.

### The ECM validity check is limited

The notebook proceeds with the short-run model when `Euribor_1M` survives long-run selection. This condition alone does not establish a valid ECM. Cointegration, residual stationarity, coefficient stability, and the significance and sign of \(\theta\) should also be assessed.

---

## Important Implementation Note: Out-of-Sample ECM Formula

The current out-of-sample ECM section is hard-coded for one particular selected model. It assumes that the long-run model contains:

- `cons`;
- `Euribor_1M`;
- `Spread`;
- `dCovid`;

and that the short-run model contains `gamma`.

The notebook itself notes that the equation must be adjusted when a different model is selected.

There are also two issues in the current prediction cell.

### 1. Long-run regressors are not all multiplied by `theta`

The intended combined equation is:

\[
r_t =
r_{t-1}(1+\theta)
-\theta\left(
\alpha+
\beta r_{t-1}^{m}+
\phi_s S_{t-1}+
\phi_c D_{t-1}^{\mathrm{COVID}}
\right)
+\gamma\Delta r_t^m
\]

The current code applies `-theta` only to the constant and Euribor terms, while adding the spread and COVID contributions outside the error-correction component.

### 2. The gamma term is not assigned to the prediction column

In the saved cell, the parenthesis closes before:

```python
+ gamma * ECM_oos["delta1_Euribor_1M"]
```

As a result, that line is evaluated as a separate expression and is not included in `customer_rate_predicted_ECM`.

A corrected version for the saved symmetric specification is:

```python
ECM_oos["customer_rate_predicted_ECM"] = (
    ECM_oos["customer_rate_lag1"] * (1 + theta)
    - theta * (
        alfa
        + beta * ECM_oos["Euribor_1M_lag1"]
        + spread_coeff * ECM_oos["Spread_lag1"]
        + covid_coeff * ECM_oos["dCovid_lag1"]
    )
    + gamma * ECM_oos["delta1_Euribor_1M"]
)
```

For a reusable implementation, construct the long-run and short-run prediction terms dynamically from `final_model_long.params` and `final_model_short.params` rather than manually naming coefficients.

---

## Forecast Interpretation

The ECM evaluation uses the actual lagged customer rate:

```text
customer_rate_lag1
```

Consequently, it is a **conditional one-step-ahead forecast**, not a fully recursive multi-period forecast.

For a recursive forecast, each predicted customer rate must become the lagged customer rate used in the next forecast period.

---

## Disclaimer

This project is intended for statistical modelling, model-development, and educational purposes. It is not financial advice and should not be used for pricing, risk measurement, behavioural modelling, or regulatory decisions without independent validation, appropriate data-governance controls, and methodological review.
