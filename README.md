# Ames Housing Price Regression

A multiple linear regression analysis of residential sale prices in Ames, Iowa, built around rigorous model selection, diagnostic checking, and out-of-sample validation rather than raw predictive accuracy.

## Overview

This project asks a simple question with a careful answer: which structural and locational characteristics of a home are associated with its sale price, and does construction quality still matter once size and location are held fixed? Using 2,930 residential sales recorded by the Ames, Iowa City Assessor's Office (2006–2010), a linear model is built from the ground up — proposal model, transformation diagnostics, outlier/influence screening, backward AIC selection, and 10-fold cross-validation — with every step checked against the standard OLS assumptions (linearity, constant variance, normality, independence).

The emphasis throughout is interpretability: the goal is to explain *how* price relates to living area, basement area, garage size, lot area, age, overall quality, and neighborhood, with defensible coefficient estimates and confidence intervals, not to squeeze out the lowest possible prediction error.

## Key results

- **Final model fit:** R² = 0.917, adjusted R² = 0.916 (F = 731.8 on 41 and 2,723 df, p < 0.001), versus adjusted R² = 0.907 for a comparable baseline model fit on the same cleaned data.
- **Model selection:** backward elimination by AIC, evaluated at every step against AIC/AICc/BIC and re-validated with a held-out test set (RMSE ≈ $20,200) and 10-fold cross-validation.
- **Diagnostics-driven transformations:** a Box–Cox transformation of sale price and a single, residual-motivated `log(1 + Garage_Area)` transformation resolved the non-constant variance and skewed residuals present in the raw model.
- **Robustness checks:** 165 outlying, high-leverage, or influential observations were identified (via standardized residuals, leverage, and Cook's distance) and excluded from the final fit; all numeric predictors carry VIF < 5, ruling out serious multicollinearity.
- **Substantive finding:** overall quality is a statistically significant, monotonically increasing predictor of price (p < 0.001 at every level) even after controlling for size and location — quality is not merely a proxy for square footage.

Full detail — coefficient tables, confidence intervals, residual plots, and a discussion of limitations — is in [`report/Ames_Housing_Regression_Report.pdf`](report/Ames_Housing_Regression_Report.pdf).

## Data

- **Source:** [`AmesHousing`](https://cran.r-project.org/package=AmesHousing) R package (Kuhn, 2020), compiled from De Cock (2011).
- **Scope:** 2,930 arm's-length residential sales in Ames, Iowa, 2006–2010.
- **Response:** `Sale_Price` (USD).
- **Predictors considered:** above-grade living area, total basement area, garage area, lot area, full bathrooms, bedrooms above grade, fireplaces, age of house, total rooms above grade, overall quality (ordinal), neighborhood (28 levels).

## Methodology

1. **Proposal model** — a hypothesis-driven six-predictor OLS model, checked for assumption violations.
2. **Expanded model** — added lot area, bathrooms, bedrooms, fireplaces, and room count as structural controls.
3. **Response transformation** — Box–Cox on `Sale_Price` to correct heteroscedasticity and skew.
4. **Predictor transformation** — `log(1 + Garage_Area)`, chosen from residual curvature (avoids the undefined `log(0)` for no-garage homes).
5. **Influence screening** — removed observations flagged as bad-leverage, outlying, or influential (Cook's distance).
6. **Backward elimination (AIC)** — iteratively dropped the term whose removal most improved AIC, tracking adjusted R², AIC, AICc, and BIC at each step.
7. **Final diagnostics** — VIF, partial F-tests, and the full residual-diagnostic panel on the selected model.
8. **Validation** — 80/20 train-test split plus 10-fold cross-validation, compared against the un-selected baseline model.

## Repository structure

```
.
├── README.md
├── LICENSE
├── report/
│   └── Ames_Housing_Regression_Report.pdf   # full written report
└── analysis/
    └── ames_housing_regression.Rmd          # complete R Markdown analysis
```

## Reproducing the analysis

Requires R (≥ 4.2) with the following packages:

```r
install.packages(c("AmesHousing", "dplyr", "ggplot2", "MASS", "knitr"))
```

Then render the full analysis:

```r
rmarkdown::render("analysis/ames_housing_regression.Rmd")
```

## Limitations

This is an observational, cross-sectional analysis — associations, not causal effects — restricted to one housing market over a five-year window, so results should not be assumed to generalize to other cities or time periods. Neighborhood-level estimates for sparsely represented areas carry wide confidence intervals. See the report's *Limitations* and *Future Improvements* sections for a full discussion, including omitted-variable considerations (school quality, income, interest rates) and possible extensions (interaction terms, non-linear models, multi-city data).

## Tech stack

R, base graphics / `ggplot2`, `MASS` (Box–Cox), `dplyr`, R Markdown / `knitr`.

## Authors

Bowen Zhao · Ge Fang · Yuyue Jiang · Yicheng Zhong · Tianyu Huang

## References

- De Cock, D. (2011). Ames, Iowa: Alternative to the Boston housing data as an end-of-semester regression project. *Journal of Statistics Education*, 19(3).
- Kuhn, M. (2020). *AmesHousing*: Ames Iowa housing data (R package version 0.0.4). CRAN.
- Yang, X. (2025). Research on house price prediction based on machine learning. *ITM Web of Conferences*, 70, 02018.
- Aziz, A., Anwar, M. M., & Dawood, M. (2021). The impact of neighborhood services on land values. *GeoJournal*, 86(4), 1915–1925.
- Chau, K. W., & Chin, T. L. (2003). A critical review of literature on the hedonic price model. *International Journal for Housing Science and Its Applications*, 27(2), 145–165.
