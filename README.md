# Direct Mail Targeting Pipeline

End-to-end Lakehouse pipeline that automatically identifies 
customers for direct mail targeting — ingesting customer, order, 
and email engagement data, applying suppression logic, and 
outputting a ranked target list ready for fulfilment.

Built in Databricks using Python, SQL, and Delta Lake following 
medallion architecture (Bronze → Silver → Gold).

---

## The problem

At a direct mail agency serving 30+ e-commerce clients, identifying 
which customers to target for direct mail was done manually — 
exporting data from different systems and cross-referencing in 
spreadsheets. This pipeline automates that process entirely.

---

## What it does

1. Ingests customer, order, and email engagement data into 
   Delta tables (Bronze layer)
2. Joins customers to email events and flags who has never 
   opened an email (Silver layer)
3. Applies three suppression filters and ranks output by 
   customer spend (Gold layer)
4. Outputs a ranked target list ready for direct mail fulfilment

---

## The targeting logic

A customer qualifies for direct mail when they meet all three conditions:
```sql
WHERE has_opened_email = false   -- never opened a single email
  AND orders_count >= 1          -- real buyer, not just a subscriber
  AND email_consent = 'subscribed' -- legal consent to be contacted
ORDER BY total_spent DESC        -- highest value customers first
```

---

## Architecture
```
Data sources
├── customers.csv    (mirrors Shopify customer data)
├── orders.csv       (mirrors Shopify order data)
└── email_events.csv (mirrors Klaviyo email engagement data)
        │
        ▼
Bronze layer — raw ingestion
├── bronze.raw_customers
├── bronze.raw_orders
└── bronze.raw_email_events
        │
        ▼
Silver layer — cleaned and joined
└── silver.dim_customers_with_email_status
    (has_opened_email flag added via LEFT JOIN)
        │
        ▼
Gold layer — final output
└── gold.mart_dm_target_audience
    (12 customers, ranked by total spend)
```

---

## Results

| Metric | Value |
|---|---|
| Customers ingested | 20 |
| Orders processed | 49 |
| Email events ingested | 15 |
| Customers qualified as DM targets | 12 |
| Suppression rate | 40% excluded as email engaged |
| Top target | Lucy Scott — £210 total spend |

---

## Tech stack

| Tool | Purpose |
|---|---|
| Databricks | Cloud notebook environment and compute |
| Delta Lake | Reliable storage format — Bronze tables |
| Python | Data ingestion notebooks |
| SQL | Transformation logic — Silver and Gold layers |
| Medallion architecture | Bronze → Silver → Gold data organisation |

---

## Project structure
```
dm_targeting/
├── notebooks/
│   ├── 01_ingest_data.ipynb      Bronze layer ingestion
│   └── 02_silver_gold_layer.sql  Silver and Gold transformations
├── data/
│   ├── customers.csv
│   ├── orders.csv
│   └── email_events.csv
└── README.md
```

---

## How to run

1. Upload the three CSV files in `/data` to a Databricks volume
2. Run `01_ingest_data.ipynb` to create Bronze Delta tables
3. Run `02_silver_gold_layer.sql` to build Silver and Gold layers
4. Query `gold.mart_dm_target_audience` for the final target list

---

## Background

This project recreates the customer targeting workflow used at a 
direct mail agency serving 30+ e-commerce clients. In that role 
I co-designed the data model alongside engineers and built the 
analytics layer clients used in real time. This project adds the 
full engineering layer from raw ingestion to ranked output — 
giving end-to-end ownership of the workflow.
