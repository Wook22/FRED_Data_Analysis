# Walmart Revenue Forecasting: Can Retail Sales Beat the Naive Baseline?

## Question
Does the Federal Reserve's monthly retail-sales index (RSXFS) improve forecasts of Walmart quarterly revenue growth when compared to a simple persistence baseline?

## Methodology

**Data & Aggregation**
- Monthly retail-sales growth (FRED) aggregated to quarterly averages; quarterly Walmart revenue sourced from company filings.
- Both series converted to year-over-year (YOY) growth to remove seasonality and focus on economic cycles.
- Data spans 2010–present with ~40–50 out-of-sample evaluation windows.

**Models Compared**
1. **Naive Baseline**: Next quarter's Walmart revenue growth = prior quarter's observed growth.
2. **Retail Signal**: OLS regression predicting Walmart YOY growth from lagged retail-sales YOY growth, retrained each quarter on expanding historical data.

**Evaluation**
- Expanding-window time-series forecasting to avoid look-ahead bias.
- Metrics: MAE, RMSE, MAPE (mean absolute percentage error).

## Findings

| Metric | Baseline | Retail Signal |
|--------|----------|---------------|
| MAE    | 0.0172   | 0.0244        |
| RMSE   | 0.0239   | 0.0336        |
| MAPE   | 1.77     | 1.40          |

**Interpretation**
- The naive baseline outperforms on absolute error metrics (MAE, RMSE), suggesting Walmart's quarterly revenue growth has strong quarter-to-quarter persistence.
- The retail-signal model performs modestly better on MAPE (20% improvement), indicating it may capture relative cyclical movements better.
- Mixed results reflect a common trade-off: a lagged feature can smooth noise but miss sudden regime shifts.

## Risks & Caveats

1. **Timing Alignment**: The forecasting model uses lagged retail-sales growth to reduce look-ahead bias. However, a production implementation would still require careful alignment with real-world data release calendars and revision timing. In practice, FRED releases preliminary retail figures weeks after month-end, and Walmart reports earnings 4–6 weeks after quarter-close. A true forward-looking forecast would require alignment with actual data-availability timelines.

2. **Structural Breaks**: Walmart's e-commerce acceleration during 2020–2021 represents a structural break not captured by pre-2020 models. Post-COVID dynamics differ materially from historical relationships.

3. **Limited Feature Set**: A single lagged predictor ignores other drivers (unemployment, interest rates, consumer confidence, gasoline prices). Additional macroeconomic and company-specific variables may improve forecast performance.

4. **Sample Size**: ~40–50 out-of-sample observations provide limited statistical power. Confidence intervals around metric differences are wide.

5. **Autocorrelation Dominance**: Strong quarter-to-quarter persistence in Walmart revenue growth makes the naive baseline difficult to beat—a sign that the dominant signal is recent momentum, not external macroeconomic factors.

## Conclusion

Retail-sales growth may contain modest incremental information about Walmart revenue growth, particularly in proportional error terms (MAPE), but the improvement is not consistent across evaluation metrics., but it does not consistently beat a simple persistence baseline. The strong performance of the naive model suggests that **Walmart's quarterly revenue is highly autocorrelated**—recent growth is the dominant predictor.

**Recommendation for Production Use**
- Deploy the naive baseline as the operational forecast.
- Use retail-sales growth as **one feature in a multivariate model** alongside seasonal terms, macro variables, and trend adjustments.
- Implement **structural-break detection** to flag COVID-era dynamics and other regime shifts.
- Establish **real-time data-availability controls** to ensure no forward-looking bias before deployment.

The analysis demonstrates prudent forecasting discipline but cautions against overfitting. Incremental gains require richer feature engineering and explicit handling of economic regimes, not model complexity.
