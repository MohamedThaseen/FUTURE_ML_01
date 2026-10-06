# FUTURE_ML_01
Sales Forecasting: Store Sales (Time Series)
Future Interns | Machine Learning Internship | Task 01

A sales forecasting project that uses historical retail data to predict the next 90 days of total sales, with model evaluation, error analysis and business-friendly visuals.

Project Overview
Accurate sales forecasts help a retailer plan inventory, staffing and procurement. In this project I built a forecasting model on grocery sales data from Corporación Favorita (Ecuador), covering January 2013 to August 2017.

Goal: forecast future daily sales and present the results in a way a non-technical business team can use.

Workflow:

1. Data cleaning and merging
  * Merged the five tables onto train using left joins (row count preserved at 3,000,888)
  * Removed duplicate holiday dates, converted dates, filled missing values
  * Oil prices were forward and back filled, and missing transactions were set to 0

2. Time-based feature engineering (on daily total sales)
  * Calendar features: year, month, quarter, day, day of week, week of year, day of year
  * Flags: weekend, month start and end, payday, national holiday
  * Trend: time index
  * Seasonality: sine and cosine of month and weekday
  * Lag features: sales 7, 14 and 28 days earlier
  * Rolling averages: previous 7 and 28 days

3. Forecasting
  * Time-based split: the last 90 days held out as the test set (no shuffling)
  * Models: Linear Regression (baseline) and Random Forest
  * 90-day future forecast generated one day at a time, feeding each prediction back in as history

4. Evaluation and error analysis
  * Metrics: MAE, RMSE, MAPE
  * Error breakdown by weekday, holidays and paydays, plus the worst-predicted days

5. Visualization
  *  Forecast chart with an uncertainty range
  *  Weekly forecast vs the same period last year
  *  Business dashboard with key numbers


Results
Evaluated on the final 90 days of data:

