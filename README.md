# CPC Bid Floor Optimization
Time series forecasting for product category cost-per-click (CPC) used to inform optimal bid floors for auctions.

## Background
In online auctions, retailers can set a bid floor, which is the minimum amount that can be bid for an ad slot. This is often arbitrary, and does not take into account actual customer bidding behavior. For example, a retailer may set a bid floor of `$0.50` across the board for all categories. However, some categories may have an average CPC bid of `$2.00` or more. Increasing the bid floor can protect revenue for these competitive categories. For other categories that are less competitive, the average CPC may be at or near the floor. Increasing the bid floor can increase revenue here, but should only be done if we have evidence that customers will pay more. Time series forecasting can be used to set optimal bid floors for both purposes.

## 1. bid_floor_ts.ipynb
This notebook contains code to:
*  Query current product categories (level 1 and 2 taxonomies), bid floors, CPC data, spend, and return on ad spend (ROAS)
*  Identify candidate categories that we are willing to change the bid floor --> "test categories"
*  Generate time series forecasts for all test categories with sufficient historical data
*  Output table of each category with the current bid floor, and upper/lower bounds of forecast confidence intervals

*Some of the code (mainly queries) and most output is redacted to protect proprietary data. However, the overall structure and logic is intact. Some output contains actual product categories, but these are publically available on retailer websites.*

**Notes**
*  Candidate categories are identified as having a high ROAS (set by `roas_lim`). This ensures that brands will still achieve good return even if spend increases as the result of bid floors increasing. Currently this is set to `400%`, i.e. a 4x return on ad spend.
*  Time series forecasts are based on monthly historical CPC data. The code loops through all categories and attempts to fit an ARIMA model for each. There is error handling to continue generating forecasts even if a category does not have enough data to forecast.
*  Output: Point forecast for each category with `90%` confidence interval. This represents the upper and lower end of CPCs that we would expect for brands to be willing to pay
*  Final data processing: remove candidate categories that have a high MAPE (high volatility), because we only want to change bid floors where we are confident with the forecasts. Additionally, set the lower bound to zero when it is negative.

Example output with random (not real) values:
| taxonomy_l1	| taxonomy_l2 | bid_floor | mape | lower_bound | est  | upper_bound |
| --- | --- | --- | --- | --- | --- | --- |
| appliances	| dishwashers | 1.17	  | 0.04	 | 1.07       | 1.97 |	3.83 |
| appliances	| microwaves  | 1.40	  | 0.06	 | 1.22       | 2.41 |  3.39 |
