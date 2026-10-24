# FitnessHub Subscription Analytics

An interactive subscription-analytics report for FitnessHub, a subscription fitness platform, built on 9,300 customers and 64,979 transactions spanning January 2024 – January 2026 (total revenue $2,594,959). The report is organized around three pages — a customer cohort matrix, a 12-month revenue forecast, and a CAC-vs-LTV break-even analysis — backed by a star-schema data model (`customers`, `transactions`, and a `dim_date` calendar table).

## Executive Summary

**1. After how many months do customers typically cancel?**
Early — the danger window is the first 90 days. Of the 9,300 customers, 4,957 (53%) had stopped paying by the end of the data window. Customers who cancelled were active for a **median of 3 months**, and **83% of all cancellations happened within the first 3 months** (mean lifetime 3.1 months). The cohort matrix shows the same story from the other side: retention falls steeply over offsets 0–3 and then flattens — customers who survive month 3 tend to stay. Retention and onboarding efforts buy the most in the first three billing cycles.

**2. What is the expected subscription revenue over the next 12 months?**
Approximately **$4.1M** if the current growth trend continues. Revenue grew from $80.6K in January 2025 to $275.6K in December 2025 (trailing-12-month total: $1.99M). A trend × seasonal-factor estimate projects February 2026 – January 2027 revenue of **$4,084,638**, rising from ~$261K/month to a December 2026 peak of ~$480K. **Yes — the forecast shows clear evidence of seasonality:** November (factor 1.14) and December (1.20) spike in both observed years, January dips after the holidays (0.94), and late spring/summer runs soft (0.94–0.96). The report's built-in forecast (12 months, seasonality 12, 95% confidence interval) tells the same story. Note the estimate assumes 2025's steep acquisition-driven growth pace continues; if acquisition slows, revenue lands nearer the bottom of the interval.

**3. How long do the 2024 and 2025 cohorts take to reach the CAC break-even point?**
- **2024 cohort** (2,816 customers, average CAC $178.10): cumulative revenue per customer crosses CAC in **month 5** (offset 4) and reaches **$429.94** to date — a 2.41× LTV-to-CAC ratio.
- **2025 cohort** (6,484 customers, average CAC $179.28): crosses CAC in **month 7** (offset 6), reaching **$213.49** so far — a 1.19× ratio that is still climbing. The slower break-even reflects the cohort's younger age mix (many members joined late in 2025), not worse unit economics; both cohorts recover acquisition cost well inside a year.

**Bottom line:** FitnessHub's economics work — acquisition cost is recovered in 5–7 months and long-lived customers are worth 2.4× their CAC — but the business leaks at the top of the lifecycle: more than half of customers churn, most of them by month 3. Moving even a fraction of month-1–3 cancellations past the 3-month cliff is worth more than equivalent acquisition spend.

## Data Model

- `customers` — one row per customer: Customer_ID, Subscription_Plan, Country, Acquisition_Channel, CAC.
- `transactions` — one row per revenue event: Transaction_ID, Customer_ID, Transaction_Date, Revenue, Transaction_Type, Product.
- `dim_date` — calendar table built with `ADDCOLUMNS(CALENDAR(...))`, including a `StartOfMonth` column used as the forecast axis.
- Relationships: `customers` 1→many `transactions` on Customer_ID; `dim_date` 1→many `transactions` on Date → Transaction_Date.

## Repository Structure

- `README.md` — this executive summary
- `report.pbix` — the interactive report (Power BI) *(added with the report build)*
- `data/Fitness_Subscriptions_Dataset.xlsx` — source data (customers + transactions)
- `screenshots/cohort_analysis.png` — cohort retention matrix
- `screenshots/revenue_forecast.png` — monthly revenue with 12-month forecast and 95% interval
- `screenshots/cac_vs_ltv.png` — cumulative LTV vs CAC for the 2024 and 2025 cohorts
- `screenshots/data_model.png` — data model diagram

## Method Notes

- A customer is treated as cancelled when their subscription payments stop before the final data month (January 2026); customers paying in the final month are counted as active. Lifetimes are measured in months with at least one subscription payment.
- Cohort = month/year of a customer's first subscription transaction. Retention at offset *k* = share of the cohort still paying *k* months after starting.
- LTV = cumulative total revenue per customer (subscriptions plus challenges, nutrition, personal training, and upgrades); CAC is the per-customer acquisition cost from the customers table, averaged per cohort.
- Forecast figures above are a transparent trend × seasonal-factor estimate computed from the data; the report itself uses the BI tool's built-in forecast with the specified settings.
