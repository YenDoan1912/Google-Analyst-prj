# Google Analytics Traffic Analysis

Exploratory analysis and dashboard for the Google Merchandise Store's web traffic data (Aug 2016 – Aug 2017), sourced from the BigQuery public GA sample dataset.

---

## Overview

This project analyzes one year of Google Analytics session data to understand traffic channel performance, user behavior, and revenue contribution. The end goal is an interactive Power BI dashboard that surfaces actionable insights for marketing and e-commerce decisions.

**Key questions answered:**
- Which traffic channels drive the most sessions and revenue?
- Which channels convert best (sessions → transactions)?
- Are there seasonal patterns in traffic and revenue?
- How do pageviews correlate with transaction likelihood?

---

## Dataset

**Source:** [`bigquery-public-data.google_analytics_sample.ga_sessions_*`](https://console.cloud.google.com/marketplace/product/obfuscated-ga360-data/obfuscated-ga360-data)

| Field | Description |
|-------|-------------|
| `date` | Session date (Aug 2016 – Aug 2017) |
| `channel` | Traffic channel (Organic Search, Direct, Referral, etc.) |
| `sessions` | Number of sessions per day per channel |
| `pageviews` | Total pageviews |
| `transactions` | Completed purchases (0 if none) |
| `revenue` | Transaction revenue in USD |

**Size:** 2,594 rows after aggregation by date × channel

---

## Project Structure

```
ga-traffic-analysis/
├── sql/
│   └── extract_traffic_by_channel.sql   # BigQuery extraction query
├── notebooks/
│   └── eda.ipynb                        # Exploratory Data Analysis
├── dashboard/
│   └── ga_dashboard.pbix                # Power BI dashboard file
├── assets/
│   └── eda_summary.png                  # EDA chart export
└── README.md
```

---

## Tools & Stack

| Layer | Tool |
|-------|------|
| Data extraction | Google BigQuery (SQL) |
| EDA | Python — pandas, matplotlib, seaborn (Google Colab) |
| Dashboard | Microsoft Power BI |

---

## BigQuery Extraction

```sql
SELECT
  PARSE_DATE('%Y%m%d', date) AS date,
  channelGrouping AS channel,
  COUNT(*) AS sessions,
  SUM(totals.pageviews) AS pageviews,
  COALESCE(SUM(totals.transactions), 0) AS transactions,
  ROUND(COALESCE(SUM(totals.transactionRevenue), 0) / 1000000, 2) AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE _TABLE_SUFFIX BETWEEN '20160801' AND '20170801'
GROUP BY date, channel
ORDER BY date
```

---

## EDA Highlights

- **Organic Search** dominates traffic (381K sessions, ~43% of total) but converts poorly
  at only 0.94% — users are in research mode, not purchase mode
- **Referral** is the highest-value channel: 5.29% conversion rate and $6.21 revenue
  per session despite mid-tier volume (104K sessions) — it drives the most total revenue ($651K)
- **Display** ads deliver the highest revenue per session at $12.51, suggesting
  high-intent buyers with large basket sizes
- **Social** generates 226K sessions (2nd largest channel) but only 0.06% conversion
  rate and $0.02 revenue per session — strong for awareness, not for direct revenue
- **Affiliates** underperform across all metrics: 16K sessions, 9 transactions, $597
  total revenue — lowest ROI of any channel
- Clear **seasonality spike** in Nov–Dec 2016 (Black Friday + holiday season), followed
  by a sharp drop in January 2017, then stable lower traffic through mid-2017
- Revenue distribution is highly skewed: mean $593/row but std $1,274 — a small number
  of high-value sessions drive a disproportionate share of total revenue ($1.54M)
---

## Power BI Dashboard

The `.pbix` file includes four report pages:

1. **Overview** — KPI cards (sessions, revenue, transactions, conversion rate) + trend lines
2. **Traffic Source** — channel breakdown by sessions, revenue, and conversion
3. **User Behavior** — pageviews per session, session depth analysis
4. **Revenue** — revenue trend, top channels by revenue per session

**DAX measures used:**
```dax
Total Sessions      = SUM(traffic_by_channel[sessions])
Total Revenue       = SUM(traffic_by_channel[revenue])
Total Transactions  = SUM(traffic_by_channel[transactions])
Conversion Rate     = DIVIDE([Total Transactions], [Total Sessions], 0) * 100
Revenue per Session = DIVIDE([Total Revenue], [Total Sessions], 0)
```

---

## How to Reproduce

1. **BigQuery** — run `sql/extract_traffic_by_channel.sql` in any GCP project (public dataset, no billing required for <1TB/month)
2. **EDA** — open `notebooks/eda.ipynb` in Google Colab, authenticate with your Google account, update `PROJECT_ID`
3. **Power BI** — open `dashboard/ga_dashboard.pbix`, refresh data source pointing to your saved BigQuery table


