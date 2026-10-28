# 📈 Sales & Demand Forecasting for Retail Businesses

A time-series forecasting project that predicts the next 90 days of sales for a multi-store grocery and retail chain, and presents the results in a business-friendly dashboard.

![Forecast dashboard](screenshots/13_forecast_dashboard.png)

---

## 📌 Business Context

Sales forecasting is one of the most widely used machine learning applications in real businesses. Companies rely on forecasts to:

- **Plan inventory**: order the right amount of stock at the right time
- **Manage cash flow**: anticipate revenue
- **Prepare staffing**: schedule people for busy and quiet days
- **Avoid overstocking or losses**: less waste and fewer empty shelves

This project shows how machine learning supports those decisions, not just how to train a model.

## 🎯 Objective

Predict future sales from historical data and present the results clearly to non-technical stakeholders. Beyond accuracy, the project focuses on:

- Preparing time-based data
- Explaining trends and seasonality
- Showing how forecasts support business planning

## 🏪 Who It's Built For

The project is framed as a case study for a multi-store grocery chain, called **"SuperKart"** in the dashboards. It uses Kaggle's public **Store Sales: Time Series Forecasting** dataset, which contains real sales data from an Ecuadorian grocery retailer. It is not a live client project.

| Stakeholder | How they use the forecast |
|---|---|
| Inventory managers | Decide how much stock to order and when |
| Finance teams | Plan cash flow from expected revenue |
| Store operations | Staff stores according to weekday demand |

---

## 📁 Dataset

**Store Sales: Time Series Forecasting** (Kaggle), made up of 7 files, 5 of which are used for modelling:

| File | Rows | Content |
|---|---|---|
| `train.csv` | 3,000,888 | Daily sales per store and product family, plus promotions |
| `stores.csv` | 54 | City, state, store type, cluster |
| `transactions.csv` | 83,488 | Daily transactions per store |
| `oil.csv` | 1,218 | Daily oil price |
| `holidays_events.csv` | 312 | Holidays and events |

The data covers **January 2013 to August 2017**. `test.csv` and `sample_submission.csv` come from the Kaggle competition and are not used.

---

## 🛠️ Tech Stack

Python · Pandas · NumPy · Scikit-learn · Statsmodels · Matplotlib · Seaborn · Jupyter / Google Colab

---

## ⚙️ Methodology

### 1. Data cleaning
- Converted dates to a common type and merged all sources onto the sales table (row count verified before and after: 3,000,888)
- Kept one holiday entry per date to avoid duplicating rows on merge
- Handled missing values: transactions filled with 0, oil price forward/back-filled over weekends and holidays, holiday fields filled with "No Holiday" / "None"
- Checked for duplicates (none found)

### 2. Time-based feature engineering
Sales were aggregated to **total daily sales** (1,688 days). Stores are closed on December 25, so those missing dates were restored with sales = 0. The final model uses 22 features:

| Group | Features |
|---|---|
| Calendar | year, month, quarter, day, day of week, week of year, day of year |
| Flags | weekend, month start/end, payday (15th and month end), national holiday |
| Trend | running time index |
| Seasonality | sine/cosine cycles for month and weekday |
| Memory | sales lags (7, 14, 28 days) and rolling averages (7, 28 days) |

Rolling averages use `shift(1)`, so a day's own sales are never used to predict that day. This avoids data leakage.

### 3. Modelling
- **Train/test split:** chronological. Training on 2013-01-29 to 2017-05-17, testing on the last **90 days** (2017-05-18 to 2017-08-15)
- **Models:** Linear Regression (baseline) and Random Forest (300 trees, max depth 12)
- **Naive benchmark:** "same weekday last week"
- **90-day forecast:** recursive. Each predicted day is fed back in to build the next day's lag features

---

## 📊 Results

| Model | MAE | RMSE | MAPE | R² |
|---|---|---|---|---|
| Naive (same day last week) | 93,935 | 122,635 | 10.47% | 0.44 |
| Linear Regression | 63,476 | 81,708 | 7.30% | 0.75 |
| **Random Forest** ✅ | **51,227** | **70,960** | **5.81%** | **0.81** |

Random Forest roughly **halves the naive baseline's percentage error** and was used for the final forecast.

![Actual vs predicted](screenshots/01_actual_vs_predicted.png)

### 90-day forecast (Aug 16 to Nov 13, 2017)

| Metric | Value |
|---|---|
| Total forecast sales | ~77.5M |
| Average per day | ~0.86M |
| vs same period last year | ~+12% (approximate) |
| Busiest day | Sun 1 Oct (~1.24M) |
| Quietest day | Thu 24 Aug (~0.65M) |

The last-year comparison is approximate because holidays and conditions differ year to year, and the first and last weeks are partial. The shaded 95% range in the dashboard comes from the test-period errors.

---

## 💡 Business Insights

- **Weekends drive the business.** Saturday and Sunday are the busiest days, with Sunday highest on average and Thursday lowest. Stock and staffing should follow this pattern.
- **Grocery and Beverages lead** product-family sales by a wide margin, followed by Produce and Cleaning.
- **Store types A and D** generate the most sales, and **Quito** is by far the largest city by sales.

| Weekday pattern | Top product families |
|---|---|
| ![Weekday](screenshots/06_sales_by_weekday.png) | ![Families](screenshots/09_top_families.png) |

### Error analysis
- 7 of the 10 largest misses were on **Sundays**, the highest-volume days, so errors there matter most in absolute terms.
- Average percentage error by weekday ranges from about 3% (Monday) to 8% (Thursday).
- The model is best on regular weekdays and weakest on peak weekend days and around holidays.

![Error analysis](screenshots/12_error_analysis.png)

---

## 🚀 Getting Started

```bash
git clone <your-repo-url>
cd <your-repo-name>
pip install pandas numpy matplotlib seaborn scikit-learn statsmodels
```

1. Download the dataset from Kaggle (*Store Sales: Time Series Forecasting*) and place the CSV files in a `data/` folder.
2. Open `Sales_Demand_Forecasting.ipynb` in Jupyter or Google Colab.
3. The notebook was written for Colab and reads a zipped dataset from Google Drive (`drive.mount(...)`). For local use, replace the loading cells with:
   ```python
   train = pd.read_csv('data/train.csv')
   stores = pd.read_csv('data/stores.csv')
   oil = pd.read_csv('data/oil.csv')
   holidays = pd.read_csv('data/holidays_events.csv')
   transactions = pd.read_csv('data/transactions.csv')
   ```
4. Run all cells. The final cell saves `sales_forecast_dashboard.png`.

---

## ⚠️ Limitations & Next Steps

- The forecast is **recursive**, so errors can compound over 90 days and the true uncertainty is wider than the shaded range.
- Forecasts are for **total daily sales**. Per-store and per-product-family forecasts would be more actionable for inventory planning.
- Future holidays are approximated by reusing month-day pairs of past national holidays, so moving holidays (e.g. Carnival, Easter) are not captured precisely.
- Evaluation uses a **single 90-day holdout**. Time-series cross-validation (`TimeSeriesSplit`) would give a more robust error estimate.
- Classical time-series models (SARIMAX, Holt-Winters) are imported in the notebook but not yet compared. They are natural next experiments, along with Gradient Boosting.

---

## 📂 Repository Structure

```
├── Sales_Demand_Forecasting.ipynb     # Full notebook (cleaning, features, modelling, dashboard)
├── README.md
├── screenshots/                       # Charts used in this README
└── data/                              # Kaggle dataset files (not included; download separately)
```

## 📜 License & Credits

Dataset: Kaggle *Store Sales: Time Series Forecasting* (check the competition page for its terms of use). Add a license of your choice (e.g. MIT) for the code before publishing.
