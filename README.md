# Olist E-Commerce Analytics Pipeline

An end-to-end SQL + Python project using the Olist Brazilian E-Commerce dataset (~100K orders across 9 relational tables). I cleaned and modeled the data in Pandas, loaded it into a SQLite database, wrote multi-table SQL queries to dig into real business questions, and built a Tableau dashboard to visualize the results.

I built this project to round out my portfolio with SQL and data engineering work — my other projects lean more ML-heavy, so I wanted something that showed relational data modeling and business-facing analysis instead.

## Business Questions

- What drives late deliveries, and where are they concentrated?
- Which product categories underperform on customer reviews?
- Which regions generate the most value per customer?

## Key Findings

- **8.11% of delivered orders arrived late.** Late deliveries are heavily concentrated in northern/northeastern Brazilian states (AL: 23.9%, MA: 19.7%, PI: 16.0%), compared to the seller-dense southeast (SP: 5.9%, MG: 5.6%). That pattern points to a logistics/distance issue rather than a general fulfillment problem.
- **Several of the worst-performing states for delivery are also among the highest in average revenue per customer** (AL, PI, MA all show up on both lists). That's a real business finding, not just a service issue — these are high-value customers being underserved, so fixing logistics there is also a retention play.
- **`office_furniture` is a consistent, high-volume underperformer on reviews** — 3.49 average across 1,687 reviews, which is a large enough sample to trust. A few other furniture categories show the same pattern, which made me think the issue is more likely shipping damage or assembly difficulty than a one-off.

Full SQL queries and analysis are in the notebook: [`notebooks/01_explore_data.ipynb`](notebooks/01_explore_data.ipynb)

## Dataset

[Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) from Kaggle — 9 CSVs covering orders, customers, order items, products, payments, reviews, sellers, geolocation, and a category name translation table, from Olist's marketplace between 2016-2018.

Raw CSVs aren't included in this repo. To run this yourself, download the dataset from the link above and drop the CSVs into a `data/raw/` folder.

## A Few Data Modeling Things I Ran Into

- **`customer_id` vs. `customer_unique_id`** — this one tripped me up at first. Olist generates a brand new `customer_id` for every single order, even for repeat customers. `customer_unique_id` is the one that actually tracks a real person across all their orders, so any customer-level (not order-level) analysis has to use that column instead.
- **`orders` to `order_items` is one-to-many** — one order can have multiple line items, so a naive join multiplies rows. Had to be careful with this when calculating anything at the order level.
- **`order_payments` can also be one-to-many per order** — some orders were split across multiple payment methods (like part gift card, part credit card), each showing up as its own row.
- **Missing data, handled two different ways** — missing `product_category_name` values got filled with `"unknown"` instead of dropped, since dropping them would've silently removed their associated orders and reviews from the analysis too. Missing delivery date columns, on the other hand, got left as `NULL` — there's no honest placeholder for a date that never happened, so I just filtered those out specifically when calculating delivery timing.

## Tech Stack

- **Python / Pandas** — cleaning and transforming the data
- **SQLite** — relational database, loaded with `to_sql()`
- **SQL** — joins, aggregations, and the actual analysis queries
- **Tableau Public** — dashboard and visualizations

## Visualizations

**Late Delivery Rate by State**
![Late Delivery Rate by State](screenshots/delivery_delay_by_state.png)

**Average Revenue per Customer by State**
![Average Revenue per Customer by State](screenshots/customer_value_by_state.png)

**Average Review Score by Product Category**
![Average Review Score by Product Category](screenshots/review_score_by_category.png)

**Combined Dashboard**
![Dashboard Overview](screenshots/dashboard_overview.png)

## What I'd Add With More Time

- A repeat purchase / retention analysis using `customer_unique_id`
- Breaking delivery delay down further — is it carrier handoff time or last-mile delivery that's driving the lateness? (Both timestamps exist in `orders`.)
- Publishing the dashboard live on Tableau Public instead of just screenshots

## Project Structure

```
olist-ecommerce-analytics-pipeline/
├── README.md
├── notebooks/
│   └── 01_explore_data.ipynb
├── screenshots/
└── tableau/
    └── Book1.twb
```
