# Prompt Log & LLM Usage Summary

## Purpose
This file documents the role of AI assistance in this forecasting analysis and reflects on human decision-making in the workflow.

## Prompts Used

### Scaffolding & Setup
1. Created submission folder structure and placeholder files based on README requirements.
2. Generated pandas boilerplate for data loading, inspection, and aggregation.
3. Drafted matplotlib plotting templates for time-series and scatter visualizations.

### Core Analysis
4. Built expanding-window forecasting loop structure using statsmodels OLS.
5. Implemented error metric calculations (MAE, RMSE, MAPE) with null-safe handling.
6. Debugged DataFrame alignment issues during quarterly merge (retail sales vs. Walmart revenue).
7. Fixed matplotlib empty-plot rendering by explicitly constructing figure and axes objects.

### Documentation
8. Generated markdown sections for exploratory analysis interpretation.
9. Drafted executive memo structure and revised for business audience.
10. Created forecast-results interpretation paragraphs.

## Key Manual Decisions & Constraints

**Forecasting Methodology**
- Rejected random train/test splits in favor of expanding-window evaluation to respect time-series dependencies.
- Introduced `yoy_retail_lag1` to avoid look-ahead bias; the assistant initially suggested same-quarter variables.
- Chose simple OLS over ensemble or ML models due to small sample size (~40–50 out-of-sample observations) and overfitting risk.

**Interpretation**
- Resisted overstating retail-sales predictive value despite modest MAPE improvement.
- Explicitly flagged structural breaks (COVID-19), timing misalignment (data-release lags), and autocorrelation dominance as reasons the naive baseline was difficult to beat.
- Framed mixed results (better MAPE, worse MAE/RMSE) as honest trade-off, not selective metric cherry-picking.

**Validation & Quality Assurance**
- Hand-verified expanding-window loop logic to confirm no data leakage.
- Checked metric calculations against manual spot-checks on subset of forecasts.
- Ran exploratory plots to visually confirm forecast reasonableness during stable vs. volatile periods.
- Reviewed markdown interpretations for technical accuracy and absence of unsupported claims.

## Reflection

The LLM was most valuable for **boilerplate generation and debugging**—matplotlib object construction, pandas merge semantics, and statsmodels API details. Less helpful for **conceptual decisions**: avoiding look-ahead bias, selecting appropriate baselines, and interpreting mixed results required domain knowledge and skepticism. One important correction involved replacing contemporaneous retail growth with lagged retail growth after recognizing the original specification could introduce look-ahead bias.

The key discipline was **constraining the model to simplicity**. Early suggestions occasionally leaned toward more complex modeling approaches than the dataset justified; time-series forecasting with small samples demands parsimony. All major methodological choices—expanding windows, lagged features, simple regression, honest caveats—were human-driven. The assistant helped implement and refine these decisions, but the methodological direction and validation remained manual.

Overall, the workflow used AI primarily for implementation support while keeping methodological decisions and validation manual.
