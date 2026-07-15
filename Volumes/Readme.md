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

#### `TFR10283`

Used for the number of current accounts in Italy.

### Manual input files

Four additional files (provided in the repository) must be placed in the notebook’s working directory:

```text
ita_10y.csv
ger_10y.csv
euribor1M.csv
rate_NMD.xlsx
```
---

## Usage

1. Clone or download the repository.
2. Place the four manually supplied market/customer-rate files beside the notebook.
3. Start Jupyter Notebook.
4. Open the notebook.
5. Run all cells from top to bottom.
6. Review the downloaded Banca d’Italia files, merged dataset, regression output, residual diagnostics, stable/core split, decay profile, and mean life.

The Banca d’Italia download step overwrites the local ZIP files with the most recently retrieved versions.

---

## Saved Illustrative Results

The outputs stored in the supplied notebook cover January 2011 through December 2025.
The final selected model retains only one-month Euribor.

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

---

## Regulatory Interpretation

The notebook applies the following assumptions from the EBA IRRBB standardised-approach framework:

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
