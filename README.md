# Corporate Loan Pricing with Machine Learning

Predicting the all-in drawn spread of U.S. syndicated corporate loans (7,605 facilities, 1996–2019) from loan terms, borrower financials, equity-market history and macro conditions. The project compares OLS, Ridge and gradient boosting against a constant benchmark using a strict chronological train / validation / test design.

**Author:** Thanh Vy Vo

## Key results

| Model | Validation RMSE (bps) | Test RMSE (bps) | Test MAE (bps) | Test R²<sub>OOS</sub> |
|---|---:|---:|---:|---:|
| Constant benchmark | 95.1 | 74.7 | 62.2 | 0.000 |
| OLS | 71.9 | 54.4 | 40.6 | 0.471 |
| Ridge (α = 0.01) | 71.9 | 54.4 | 40.6 | 0.471 |
| **Gradient boosting** (selected on validation) | **70.9** | 60.4 | 46.5 | 0.347 |

- The validation-selected model (gradient boosting) cuts test RMSE by 19% against the benchmark. The borrower-clustered bootstrap 95% CI for the improvement excludes zero.
- On the later test years the linear models rank first. The validation margin was only 1.4%, so the choice of model does not carry over cleanly from one period to the next.
- Loan and industry terms carry the most predictive information, followed by borrower financials. Market history and macro variables add little.

## How to read this project

| If you have… | Read |
|---|---|
| 5 minutes | [`loan_pricing_report.pdf`](loan_pricing_report.pdf): a 5-page summary of the findings, figures and limitations, with no code |
| 30 minutes | [`loan_pricing_analysis.ipynb`](loan_pricing_analysis.ipynb): the full analysis with code and outputs. It renders on GitHub, and the outputs are already saved, so you don't need to run it |
| Specific numbers | [`results_summary.json`](results_summary.json): every metric the notebook computes, as machine-readable output |

The notebook has five sections:

1. **Inspect the data**: sample coverage, spread distribution, missingness
2. **Split and prepare**: chronological splits, leakage-safe preprocessing fitted on training data only
3. **Estimate and select models**: OLS, Ridge and gradient boosting tuned on validation RMSE, plus nested information-block comparisons
4. **Evaluate later loans**: a single test evaluation of the frozen model, error breakdowns by year and borrower type
5. **Interpret**: permutation importance, partial dependence, bootstrap comparisons, limitations

## Methodology highlights

- **Chronological splits.** Training covers 1996–2014, validation 2015–2017 and test 2018–2019. Loan packages never cross splits.
- **No leakage.** Imputation, caps, scaling and encoding are fitted on the fitting sample only. The test targets stay locked in code until the model choice is frozen, and the test set is scored exactly once.
- **Honest uncertainty.** Model differences come with borrower-clustered bootstrap intervals. Out-of-sample R² uses the train+validation mean as its benchmark, and it is split into level error and dispersion error.
- **Reproducible.** Every random step uses a fixed seed (42). Re-running the notebook reproduces `results_summary.json` byte for byte.

## Data

| Source | Content |
|---|---|
| DealScan | Facility terms and the target `spread_bps` (all-in drawn spread, including fees) |
| Compustat | Borrower financials, lagged 6–18 months before origination |
| CRSP | 12-month stock return and volatility before origination |
| FRED | CPI inflation, unemployment and the federal funds rate, lagged two months |

Variable definitions are in [`data/data_dictionary.csv`](data/data_dictionary.csv). The loan-level dataset comes from licensed WRDS sources, so **it is not included in this repository**.

## Reproduce

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# place Corporate_Loan_Pricing_1996_2019.csv in data/
jupyter nbconvert --to notebook --execute --inplace loan_pricing_analysis.ipynb   # ~1 min; writes figures/ and results_summary.json
tectonic report/loan_pricing_report.tex --outdir .                               # optional: rebuild the PDF report
```

## Repository layout

```
├── loan_pricing_analysis.ipynb   # main analysis (executed)
├── loan_pricing_report.pdf       # 5-page written report
├── results_summary.json          # all computed metrics
├── figures/                      # charts produced by the notebook
├── report/loan_pricing_report.tex
├── data/data_dictionary.csv      # variable definitions (dataset not included)
└── requirements.txt
```
