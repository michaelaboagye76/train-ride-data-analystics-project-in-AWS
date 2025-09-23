---

# Train Rides Data Analytics Project

## Project Overview

This project focuses on analyzing train ticket sales, pricing, and operational performance using AWS Glue, Athena, and S3.
Raw ticket transaction data in CSV format was stored in **Amazon S3**, cleaned using **AWS Glue DataBrew**, cataloged using an **AWS Glue Crawler**, and queried with **Amazon Athena** for insights.

The goal was to:

* Standardize and clean raw ticket data.
* Structure it in the **Glue Data Catalog**.
* Run SQL queries in **Athena** to generate insights.
* Prepare data for visualization in **QuickSight** or other BI tools.

---

##  Data Description

The dataset contained transaction-level train ticket information with the following key fields:

* **Transaction & Purchase**

  * `"transaction id"` (string)
  * `"date of purchase"` (string → converted to `DATE`)
  * `"time of purchase"` (string → converted to `TIMESTAMP`)
  * `"purchase type"` (e.g., online, station)
  * `"payment method"` (card, cash, etc.)

* **Ticket Details**

  * `"railcard"` (discount eligibility)
  * `"ticket class"` (first, second, etc.)
  * `"ticket type"` (single, return, season pass)
  * `"price"` (double)

* **Journey Details**

  * `"departure station"` / `"arrival destination"`
  * `"date of journey"` (string → `DATE`)
  * `"departure time"`, `"arrival time"`, `"actual arrival time"` (timestamps)
  * `"journey status"` (on-time, delayed, cancelled)
  * `"reason for delay"`
  * `"refund request"`

---

## Architecture Diagram
![](images/architectire_diagram.png)

##  Data Engineering Workflow

1. **Data Storage**

   * Raw CSVs uploaded to **Amazon S3** (`s3://trainridesdatabucketreeu76/raw/`).

2. **Data Cleaning (AWS Glue DataBrew)**

   * Replaced `NULL` values with `"Unknown"` or `"N/A"`.
   * Standardized text (trim spaces, lowercased values).
   * Converted date/time fields (`"date of purchase"`, `"date of journey"`, `"departure time"`) from `string` → `DATE`/`TIMESTAMP`.
   * Dropped irrelevant or duplicate columns.
   * Saved cleaned datasets back into S3 (`s3://trainridesdatabucketreeu76/clean/`).

3. **Schema Cataloging (AWS Glue Crawler)**

   * Crawler scanned the S3 bucket.
   * Generated schema in **Glue Data Catalog** (`trainridesdb`).
   * Created a queryable table in Athena.

4. **Querying & Transformation (Athena)**

   * Cast date/time strings into proper types using `CAST` and `date_parse`.
   * Aggregated ticket sales, prices, and journey performance.

5. **Visualization (BI-ready)**

   * Data was prepared for dashboards in **Amazon QuickSight** (or Power BI/Tableau).

---

##  Key Insights Generated

1. **Revenue & Pricing**

   * Average ticket price by **ticket class** and **departure station**.
   * Top 10 **routes by total revenue**.
   * Revenue breakdown by **payment method**.

2. **Operational Efficiency**

   * Average **delay times by station**.
   * Most common **reasons for delay**.
   * Refund requests distribution by **ticket type**.

3. **Customer Behavior**

   * Railcard usage by **ticket class**.
   * Peak **purchase times** (morning vs evening).
   * Seasonal demand trends by **date of purchase**.

---

##  Sample Queries

**Average Price by Departure Station**

```sql
SELECT
    "departure station",
    AVG(price) AS avg_price
FROM trainridesdb."train_tickets"
GROUP BY "departure station"
ORDER BY avg_price DESC;
```

**Top Routes by Revenue**

```sql
SELECT
    "departure station",
    "arrival destination",
    SUM(price) AS total_revenue
FROM trainridesdb."train_tickets"
GROUP BY "departure station", "arrival destination"
ORDER BY total_revenue DESC
LIMIT 10;
```

**Refund Requests by Ticket Type**

```sql
SELECT
    "ticket type",
    COUNT("refund request") AS refund_count
FROM trainridesdb."train_tickets"
WHERE "refund request" IS NOT NULL
GROUP BY "ticket type";
```

---

## Deliverables

* **Cleaned dataset** in S3 (`/clean/`).
* **Glue Data Catalog schema** (`trainridesdb`).
* **Athena SQL queries** for analysis.
* **Insights & visualizations** ready for BI dashboards.

---


