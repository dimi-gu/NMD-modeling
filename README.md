# Non Maturity Deposit modelling

This repository contains Python-based models for analysing the behavioural characteristics of **Non-Maturity Deposits (NMDs)** within an ALM and IRRBB framework.

The project is organised into two complementary modules:

## Customer Rate Model

The [`Customer rate`](./Customer%20rate) directory models the relationship between NMD customer rates and market rates.

It includes:

- long-run interest-rate pass-through estimation;
- short-run Error-Correction Model dynamics;
- asymmetric reactions to increasing and decreasing market rates;
- mean-reversion analysis;
- statistical diagnostics and out-of-sample testing.

## Volume Model

The [`Volumes`](./Volumes) directory estimates the stability and behavioural maturity of NMD balances.

It includes:

- modelling of deposit volumes per account;
- Minimum Probable Amount estimation;
- stable, core and non-core balance decomposition;
- long-term core-balance decay profiles;
- model-implied mean-life calculation.

## Technologies

The models are implemented in Python and use libraries including:

- `pandas`
- `numpy`
- `statsmodels`
- `scipy`
- `matplotlib`

## Disclaimer

This repository is intended for educational and methodological purposes. The models are illustrative and should not be used for regulatory reporting, risk measurement or balance-sheet management without institution-specific calibration, validation and governance.
