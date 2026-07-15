# Non Maturity Deposit volume modelling

A Python/Jupyter workflow for estimating the behavioural liquidity profile of **non-maturity deposits (NMDs)** using public Italian banking data.

The project is designed as an illustrative IRRBB behavioural-modelling example rather than a production-ready bank model.

---

## Main Features
The notebook:
1. downloads household deposit balances and the count of accounts data from Banca d’Italia;
2. combines them with Euribor, sovereign-spread, and customer-rate data;
3. models monthly growth rate in deposits per account;
4. estimates a conservative stable balance using a **Minimum Probable Amount (MPA)** approach;
5. separates balances into core and non-core components;
6. generates a long-term core-deposit decay profile;
7. calculates the model-implied mean life.

---

## Modelling Objective

The volume model is intended to estimate two behavioural characteristics of NMD balances:

- the proportion of balances considered **stable**;
- the conservative liquidity or amortisation profile of the stable/core component.

The model assumes no future new business and no rollover of maturing balances. The projected core balance therefore represents a survival or decay profile rather than a forecast of total future deposits.

---

## Data Sources

### Banca d’Italia data downloaded automatically

The notebook downloads and extracts two complete statistical cubes through the Banca d’Italia machine-to-machine service.

#### `TDB20290`

Used for household deposit amounts.

The notebook retains observations where:

```python
loc_ctp == "IT"
set_ctp in ["S14BI2", "600"]
```

The two household-sector categories are aggregated:

- producer households with up to five employees;
- consumer households.

The source values are multiplied by 1,000 because the exported amounts are expressed in thousands.

#### `TFR10283`

Used for the number of current accounts in Italy.

The notebook retains:

```python
loc_sport == "IT"
fenec == "5833004"
```

The series is reported annually and is converted to monthly frequency through linear interpolation.

Downloaded files are stored under:

```text
data/raw/bancaditalia/
├── TDB20290.zip
├── TDB20290/
├── TFR10283.zip
└── TFR10283/
```

An internet connection is required when the download cell is executed.

### Files supplied manually

Four additional files must be placed in the notebook’s working directory:

```text
ita_10y.csv
ger_10y.csv
euribor1M.csv
rate_NMD.xlsx
```

#### `ita_10y.csv`

Monthly Italian 10-year government-bond yield.

ECB series used by the notebook:

```text
IRS.M.IT.L.L40.CI.0000.EUR.N.Z
```

#### `ger_10y.csv`

Monthly German 10-year government-bond yield.

ECB series:

```text
IRS.M.DE.L.L40.CI.0000.EUR.N.Z
```

#### `euribor1M.csv`

Monthly one-month Euribor.

ECB series:

```text
FM.M.U2.EUR.RT.MM.EURIBOR1MD_.HSTA
```

Each ECB CSV must contain:

- a `DATE` column;
- one value column.

Columns containing `TIME PERIOD` or `TIME_PERIOD` are removed automatically.

#### `rate_NMD.xlsx`

NMD customer rates exported from the Banca d’Italia database.

The notebook reads the sheet:

```text
Report
```

The first column is treated as the date, and the column:

```text
Producer households
```

is renamed:

```text
customer_rate
```

Customer rates are reported quarterly and are converted to monthly frequency through linear interpolation.

---

## Suggested Repository Structure

```text
.
├── NMD_Volume_Model.ipynb
├── rate_NMD.xlsx
├── euribor1M.csv
├── ita_10y.csv
├── ger_10y.csv
├── Mean Life - Mathematica.pdf
├── data/
│   └── raw/
│       └── bancaditalia/
└── README.md
```

The `data/raw/bancaditalia` directory is created automatically by the notebook.

---

## Data Preparation

### Household deposit amount

The household categories selected from `TDB20290` are summed by observation date:

\[
V_t =
V_t^{\text{consumer households}}
+
V_t^{\text{producer households}}
\]

where \(V_t\) is the aggregate household deposit balance.

### Number of accounts

The annual number of current accounts is reindexed to a complete month-end date range and linearly interpolated.

### Deposit amount per account

The notebook defines:

\[
V_t^{pc}
=
\frac{V_t}{N_t}
\]

where:

- \(V_t\) is the total deposit amount;
- \(N_t\) is the number of accounts;
- \(V_t^{pc}\) is the average deposit amount per account.

### Target variable

The model target is the monthly log growth rate of deposits per account:

\[
g_t
=
\log(V_t^{pc})
-
\log(V_{t-1}^{pc})
\]

Modelling the per-account growth rate is intended to reduce structural effects caused by changes in the number of accounts or the scope of banking operations.

### Sovereign spread

The Italian–German ten-year spread is calculated as:

\[
S_t =
Y_t^{IT,10Y}
-
Y_t^{DE,10Y}
\]

### COVID dummy

The notebook sets:

```python
dCovid = 1
```

from 28 February 2020 through 31 December 2020 and zero otherwise.

### Rate units

Before estimation, the following variables are divided by 100:

- `Euribor_1M`;
- `customer_rate`;
- `Spread`.

The regression therefore uses decimal rates rather than percentages.

---

## Volume Model

The initial OLS specification is:

\[
g_t =
\alpha
+\beta_E E_t
+\beta_C C_t
+\beta_S S_t
+\beta_D D_t
+\varepsilon_t
\]

where:

- \(g_t\) is monthly log growth in deposits per account;
- \(E_t\) is one-month Euribor;
- \(C_t\) is the NMD customer rate;
- \(S_t\) is the Italian–German sovereign spread;
- \(D_t\) is the COVID dummy;
- \(\varepsilon_t\) is the regression residual.

The initial candidate variables are:

```python
variables = [
    "Euribor_1M",
    "customer_rate",
    "Spread",
    "dCovid",
]
```

### Backward elimination

The notebook repeatedly:

1. estimates the OLS model;
2. identifies the non-intercept variable with the largest p-value;
3. removes that variable when its p-value exceeds 5%;
4. stops when every remaining variable is significant at the 5% level.

No economic-sign restrictions are imposed during selection.

---

## Saved Illustrative Results

The outputs stored in the supplied notebook cover January 2011 through December 2025.

The final selected model retains only one-month Euribor:

\[
g_t =
0.0008
-
0.1467 E_t
+\varepsilon_t
\]

Saved results include:

| Measure | Result |
|---|---:|
| Observations | 179 |
| \(R^2\) | 7.3% |
| Euribor coefficient | -0.1467 |
| Euribor p-value | < 0.001 |
| Residual standard deviation | 0.00709 |
| Durbin–Watson statistic | 2.354 |
| Stable balance | 98.36% of total deposits |
| Core balance after cap | 90% of total deposits |
| Non-core balance after floor | 10% of total deposits |
| Model-implied mean life | 5,061.7 months |
| Mean life in years | approximately 421.8 years |

The negative Euribor coefficient is interpreted as customers moving funds from deposits toward more remunerative investments when market rates rise.

These values are outputs of the saved dataset and model specification; they should not be treated as generally applicable behavioural assumptions.

---

## Residual Analysis

The notebook calculates residual volatility using the sample standard deviation.

It also applies:

- Kolmogorov–Smirnov normality test;
- Shapiro–Wilk normality test.

In the saved run, both tests reject residual normality:

| Test | Statistic | p-value |
|---|---:|---:|
| Kolmogorov–Smirnov | 0.115 | 0.016 |
| Shapiro–Wilk | 0.902 | < 0.001 |

The notebook nevertheless uses a standard-normal 1% quantile in the MPA calculations. This is an important modelling limitation discussed below.

---

## In-Sample Forecast Interval

The estimated growth rate is presented with a 1%–99% interval:

\[
\widetilde g_t
=
\widehat\mu_t
\pm
\sigma q_p
\]

where:

- \(\widehat\mu_t\) is the fitted growth rate;
- \(\sigma\) is the residual standard deviation;
- \(q_p\) is the corresponding standard-normal quantile.

The notebook uses:

```python
z_1 = norm.ppf(0.01)
z_99 = norm.ppf(0.99)
```

---

## Stable Balance: Minimum Probable Amount

The stable balance at the initial projection point is estimated as the 99% lower bound of the one-period balance distribution:

\[
\operatorname{Stable}(0)
=
V_{-1}
\exp\left(
\mu_0
+
q_{0.01}\sigma
\right)
\]

where:

- \(V_{-1}\) is the latest observed total deposit amount;
- \(\mu_0\) is the modelled drift;
- \(\sigma\) is residual volatility;
- \(q_{0.01}\) is the 1% standard-normal quantile.

The lower the estimated volatility, the larger the stable balance.

### Unconditional drift

The notebook uses an unconditional forecast assumption. The significant explanatory variables are replaced by their historical sample means:

\[
\mu =
\alpha
+
\sum_j \beta_j \overline X_j
\]

Positive drift is capped at zero:

```python
drift = min(drift_raw, 0)
```

This prevents the behavioural profile from increasing over time.

In the saved run, the capped drift is zero.

---

## Core and Non-Core Deposits

The notebook applies a long-run customer-rate pass-through parameter:

```python
beta = 0.0589
```

This parameter is taken manually from a separate customer-rate model.

The uncapped core relationship is:

\[
\operatorname{Core}
=
\operatorname{Stable}
(1-\beta)
\]

The notebook then applies:

- a core cap of 90% of total deposits;
- a non-core floor of 10% of total deposits.

The saved run therefore produces:

\[
\operatorname{Core}=90\%\times V
\]

\[
\operatorname{NonCore}=10\%\times V
\]

The non-core component is assigned an overnight maturity.

---

## Core-Balance Decay Profile

The notebook creates a long-term monthly projection and calculates the surviving core balance as:

\[
\operatorname{Core}(t)
=
\operatorname{Core}_0
\exp\left[
\mu_t^*
-\frac{\sigma^2}{2}t
+
q_{0.01}\sqrt{t}\sigma
\right]
\]

where:

- \(\mu_t^*\) is cumulative drift;
- \(\sigma\) is monthly residual volatility;
- \(q_{0.01}\) is the 1% standard-normal quantile;
- \(t\) is measured in months.

The current notebook creates 360 rows intended to represent a 30-year horizon.

---

## Mean-Life Calculation

The normalised survival function is defined as:

\[
S_v(t)
=
\exp\left[
\left(
\mu-\frac{\sigma^2}{2}
\right)t
+
q_{0.01}\sqrt{t}\sigma
\right]
\]

The corresponding density is:

\[
f_v(t)
=
-\frac{dS_v(t)}{dt}
\]

Mean life is interpreted as the expected time at which the core balance is amortised:

\[
E[t]
=
\int_0^\infty
t f_v(t)\,dt
\]

The notebook implements the following closed-form expression:

\[
E[t]
=
\frac{1}{A}
+
\frac{q\sigma\sqrt{\pi}}
{2A^{3/2}}
\exp\left(
\frac{q^2\sigma^2}{4A}
\right)
\left[
1+
\operatorname{erf}\left(
\frac{q\sigma}{2\sqrt A}
\right)
\right]
\]

where:

\[
A=
\frac{\sigma^2}{2}-\mu
\]

The calculation is valid only when:

\[
A>0
\]

The mathematical derivation is referenced in:

```text
Mean Life - Mathematica.pdf
```

---

## Installation

The notebook was saved with Python 3.13.5.

Install the required packages with:

```bash
pip install requests pandas numpy matplotlib scipy statsmodels openpyxl jupyter
```

Then start Jupyter:

```bash
jupyter notebook
```

---

## Usage

1. Clone or download the repository.
2. Place the four manually supplied market/customer-rate files beside the notebook.
3. Ensure the file and sheet names match the expected names exactly.
4. Start Jupyter Notebook.
5. Open the notebook.
6. Run all cells from top to bottom.
7. Review the downloaded Banca d’Italia files, merged dataset, regression output, residual diagnostics, stable/core split, decay profile, and mean life.

The Banca d’Italia download step overwrites the local ZIP files with the most recently retrieved versions.

---

## Outputs

The notebook produces:

1. downloaded and extracted Banca d’Italia source files;
2. filtered household deposit balances;
3. interpolated monthly account counts;
4. aggregate deposit and account-count charts;
5. merged market-rate and customer-rate dataset;
6. market-rate, customer-rate, and spread chart;
7. per-account deposit balances;
8. monthly per-account log-growth series;
9. OLS variable-elimination log;
10. final OLS regression summary;
11. residual time-series chart;
12. residual volatility estimate;
13. normality-test results;
14. fitted-versus-actual growth chart;
15. 1%–99% in-sample interval;
16. stable-balance estimate;
17. core and non-core allocation;
18. projected core-balance decay chart;
19. analytical mean-life result.

---

## Important Methodological Notes

### Interpolation materially smooths the data

The number of accounts is observed annually and linearly interpolated to monthly frequency. The customer rate is observed quarterly and is also linearly interpolated.

This creates artificial smoothness and affects:

- monthly growth volatility;
- residual distribution;
- coefficient significance;
- the MPA stable percentage;
- the speed of the decay profile;
- mean life.

The saved notebook explicitly attributes its extremely long mean life partly to this smoothing effect.

### Residual normality is rejected

Both saved normality tests reject a Gaussian residual distribution, but the MPA uses the standard-normal 1% quantile.

A production model should consider:

- an empirical residual quantile;
- bootstrap simulation;
- a fitted heavy-tailed distribution;
- filtered historical simulation;
- another validated conservative-tail methodology.

The standard Kolmogorov–Smirnov p-value is also not exact when the tested normal distribution’s mean and standard deviation are estimated from the same sample. A Lilliefors-type test is more appropriate for that setup.

### Log-growth drift and the variance correction should be aligned

The regression directly estimates:

\[
\Delta\log(V_t^{pc})
\]

This is already a log-growth measure.

The future survival equation additionally subtracts:

\[
\frac{\sigma^2}{2}t
\]

That correction is normally introduced when converting an arithmetic level-process drift into a log-process drift. If the estimated \(\mu\) is already the expected log growth, subtracting \(\sigma^2/2\) may double-adjust the drift.

The interpretation of \(\mu\) should therefore be defined consistently. Under a direct log-growth model, a natural lower-quantile specification is:

\[
V_t^{L}
=
V_0
\exp\left(
t\mu+
q_p\sigma\sqrt t
\right)
\]

Changing this assumption also changes the mean-life formula.

### The per-account model is applied to total balances

The model estimates the behaviour of deposit amounts per account but applies the resulting distribution to the latest total deposit amount.

This assumes that future account-count dynamics do not materially alter the total-balance survival profile. A production implementation should model or scenario-test the number of accounts separately.

### The numerator and denominator populations must be aligned

The numerator is an aggregate household deposit amount, while the denominator is a selected count of current accounts.

Before using this approach on another dataset, verify that:

- the instrument scope is consistent;
- the customer population is consistent;
- joint and multiple accounts are treated appropriately;
- the number of accounts is a meaningful exposure measure for the selected deposit balance.

### The regression uses contemporaneous variables

The target is monthly deposit growth, while Euribor, customer rates, the spread, and the COVID dummy enter contemporaneously.

This may introduce:

- simultaneity;
- endogeneity;
- unstable causal interpretation.

Lag structures, distributed-lag models, dynamic regressions, or a separately validated forecasting design may be more appropriate.

### Time-series properties are not tested

The notebook does not formally test:

- stationarity of the regressors;
- parameter stability;
- structural breaks;
- autocorrelation;
- heteroskedasticity;
- out-of-sample performance.

Backward elimination based only on in-sample p-values may produce an unstable specification.

### The final OLS covariance estimator is non-robust

The saved regression uses conventional OLS standard errors. Monthly time-series residuals may require HAC/Newey–West or another justified covariance estimator.

### Low explanatory power

The saved model has an \(R^2\) of 7.3%. Most monthly variation remains in the residual term.

This does not automatically invalidate the model, but it means the stability result is driven largely by the estimated residual distribution and the modelling assumptions rather than by explanatory-variable forecasts.

---

## Important Implementation Notes

### Missing `warnings` import

The notebook calls:

```python
warnings.warn(...)
```

when expected files are absent, but `warnings` is not imported in the initial import cell.

Add:

```python
import warnings
```

Otherwise, a missing input file can trigger a `NameError` instead of the intended warning.

### Pass-through is hard-coded

The value:

```python
beta = 0.0589
```

is manually copied from another model.

For reproducibility, it should be:

- loaded from a controlled model output;
- passed as a documented configuration parameter;
- or estimated within an integrated pipeline.

Its estimation date, customer segment, and confidence interval should be stored with the result.

### Non-core should be calculated as the residual balance

The current code uses:

```python
core = min(stable_mpa_value * (1 - beta), 0.9 * total_amount)
non_core = max(stable_mpa_value * beta, 0.1 * total_amount)
```

These two independently capped formulas happen to sum to total deposits in the saved run, but they are not guaranteed to do so for every stable percentage and pass-through value.

A balance-preserving implementation is:

```python
core = min(stable_mpa_value * (1 - beta), 0.9 * total_amount)
non_core = total_amount - core
```

This automatically includes:

- the non-stable component;
- the repricing portion of stable deposits;
- the effect of the core cap.

### Projection indexing is offset

The projection table assigns:

```text
t = -1, 0, 1, ..., 358
```

across 360 rows.

The first row contains the latest observed balance, the second row is treated as \(t=0\), and the final row is \(t=358\). This is shorter than a complete \(t=0,\ldots,359\) 360-month forecast.

The base observation and future horizon should be constructed explicitly, for example with one base row plus 360 projected months.

### Cumulative drift is offset when drift is non-zero

The code calculates cumulative drift before assigning the time index:

```python
DATA_future["cumulative_drift"] = DATA_future["drift"].cumsum()
```

Because the table includes the base row and the \(t=0\) row, cumulative drift at later horizons is offset by additional periods.

This has no effect in the saved run because drift is zero. For non-zero drift, calculate cumulative drift directly from the intended horizon:

```python
DATA_future["cumulative_drift"] = DATA_future["drift"] * DATA_future["t"]
```

with appropriate treatment of the base and \(t=0\) rows.

### Mean life exceeds the projection horizon

The saved mean life is approximately 422 years, while the displayed survival profile covers about 30 years.

The plot therefore does not show complete amortisation. A production workflow should reconcile:

- analytical mean life;
- projection horizon;
- residual balance at the end of the horizon;
- applicable regulatory maturity constraints.

---

## Regulatory Interpretation

The notebook applies the following assumptions from the cited EBA IRRBB standardised-approach framework:

- stable balances are separated into core and non-core components;
- non-core deposits receive overnight treatment;
- retail transactional core deposits are capped at 90% of total deposits;
- the average maturity of the core component is capped at five years.

The notebook’s model-implied mean life greatly exceeds the five-year cap. The model output should therefore be distinguished from the maturity ultimately used for regulatory measurement.

Users should verify the applicable regulation, version, scope, and institution-specific requirements before relying on these assumptions.

---

## Disclaimer

This repository is intended for educational, methodological, and model-development purposes.

It is not investment advice, regulatory advice, or a validated IRRBB production model. Results based on national aggregate public data are not representative of an individual institution’s customer behaviour. Any production use requires institution-specific data, governance, independent validation, backtesting, sensitivity analysis, and review against the applicable regulatory framework.
