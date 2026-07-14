# Credit_Card_Financial_Dashboard
Power BI Dashboard
# Credit Card Business Intelligence Dashboard

An end-to-end BI project that tracks credit card revenue, customer segmentation, and transaction behavior for a bank's card portfolio — built with **PostgreSQL, Power BI, DAX, Power Query, and Excel**.

The project is designed to mirror a real analytics workflow: raw data lands in a SQL database, gets modeled and transformed, and feeds a two-page interactive dashboard that a business team could actually use week to week — including a simulated live data refresh (see [Incremental Load](#incremental-load-simulation) below).


## Project Objective

The dashboard answers three core business questions:

- **Revenue tracking** — How much revenue is the card portfolio generating, and how is it trending week over week and quarter over quarter?
- **Customer segmentation** — Which customer segments (income, job, education, age, marital status, geography) drive the most value?
- **Transaction behavior** — How are customers actually using their cards (swipe vs. chip vs. online), and on what type of spend?


## Dataset

Two paired datasets, joined on customer ID (`Client_Num`):

| File | Rows | Grain | Key fields |
|---|---|---|---|
| `customer.csv` | 10,108 | one row per customer | age, gender, dependents, education, marital status, state, job, income, satisfaction score |
| `credit_card.csv` | 10,108 | one row per customer per week | card category, credit limit, revolving balance, transaction amount/count, utilization ratio, chip type, expense type, interest earned |
| `cust_add.csv` | 185 | new customers arriving in week 53 | same schema as `customer.csv` |
| `cc_add.csv` | 185 | week 53 transactions | same schema as `credit_card.csv` |

> The `_add` files simulate a new week of live data arriving in the system — used to test and demonstrate the incremental refresh described below.


## Steps / Approach

1. **Data staging (PostgreSQL)** — loaded `customer.csv` and `credit_card.csv` into base tables; cleaned data types, date formats, and inconsistent text fields
2. **Data modeling** — built a one-to-many relationship between the customer and transaction tables in Power BI
3. **Power Query transformations** — reshaped data on load: fixed date formats, derived columns (age group, income group, quarter)
4. **DAX measures** — built the calculation layer: total revenue, week-over-week % change, quarter totals, running totals
5. **Transaction Report page** — revenue by card category, expense type, quarter, and chip type; weekly revenue trend and WoW comparison table
6. **Customer Report page** — revenue sliced by demographics: age, income group, job, education, marital status, state, dependent count, gender
7. **Interactivity** — slicers for quarter, week, card category, and chip usage filtering both pages
8. **Incremental load simulation** — inserted `cust_add.csv` / `cc_add.csv` into the database and refreshed Power BI to confirm all visuals update without any redesign (see below)
9. **Publishing** — published to Power BI Service for a shareable link; exported static views for the portfolio


## Incremental Load Simulation

The `sql/week53_incremental_load.sql` script simulates a new week of data landing in the system:

1. Stages `cust_add.csv` and `cc_add.csv` into temporary tables
2. Runs data-quality checks (duplicate customer+week rows, orphaned customer IDs)
3. Merges new customers and new transaction rows into the main tables
4. Validates the load (row counts, max week)
5. Power BI is refreshed and every measure/visual (WoW revenue, weekly trend, quarter totals) updates automatically — proof the DAX layer is built off relative week logic rather than hardcoded values


## Key Insights

- **Revenue is heavily concentrated in the Blue tier** — ~$92.3M of the $110.6M total (≈83%) comes from Blue-category cards alone, with Silver, Gold, and Platinum splitting the remainder.
- **Spend skews toward everyday essentials** — Bills ($28M), Entertainment, and Fuel (~$19M each) lead the expense-type breakdown; discretionary travel spend is the smallest category (~$12M).
- **Swipe still dominates transaction methods** — $70M of revenue comes from swipe vs. $34M chip and just $7M online, suggesting lagging contactless/online adoption.
- **Education and income correlate with revenue concentration** — Graduate-level customers contribute ~$45M (≈40%) of total revenue; high-income customers account for roughly half.
- **Revenue is flat across quarters** (~27-28M each) — no seasonal spike, suggesting a stable but non-growing portfolio.


## Repository Structure

```
credit-card-bi-dashboard/
├── README.md
├── data/
│   ├── customer.csv
│   ├── credit_card.csv
│   ├── cust_add.csv
│   └── cc_add.csv
├── sql/
│   └── week53_incremental_load.sql
├── powerbi/
│   └── credit_card_dashboard.pbix
└── screenshots/
    ├── transaction_report.png
    └── customer_report.png


## Tech Stack

`PostgreSQL` · `Power BI` · `DAX` · `Power Query` · `Excel`

---

## How to Reproduce

1. Load `data/customer.csv` and `data/credit_card.csv` into a PostgreSQL database
2. Open `powerbi/credit_card_dashboard.pbix` and point the data source to your database
3. Refresh — the two report pages will populate
4. To test the incremental load: run `sql/week53_incremental_load.sql` against your database, then refresh Power BI again and confirm week 53 appears across all visuals
