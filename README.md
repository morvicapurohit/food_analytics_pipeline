# Food Analytics Data Pipeline

End-to-end data pipeline that extracts semi-structured food product data from MongoDB, processes it through Apache Spark, builds a relational analytics schema in DuckDB, and runs business intelligence queries across sales, reviews, and supplier dimensions.

---

## Problem Statement

Food companies need to understand how product nutritional profiles (like NutriScore grades) relate to sales performance, customer satisfaction, and supplier reliability. The challenge: product data lives in a semi-structured NoSQL database, while sales and reviews are in flat files — requiring a pipeline that cleans, joins, and structures everything for analysis.

---

## Architecture

```
MongoDB (raw product data)
    │
    ▼
Apache Spark (extract, clean, flatten nested fields)
    │
    ▼
Parquet files (intermediate storage)
    │
    ▼
DuckDB (relational analytics layer)
    │
    ├── Star Schema
    │   ├── products (dimension)
    │   ├── sales (fact)
    │   ├── reviews (fact)
    │   ├── suppliers (dimension - extension)
    │   └── product_supplier (bridge)
    │
    └── Analytical Queries → Business Insights
```

---

## Star Schema Design

```
                    ┌──────────────┐
                    │  suppliers   │
                    │──────────────│
                    │ supplier_id  │
                    │ supplier_name│
                    │ country      │
                    └──────┬───────┘
                           │
┌──────────────┐   ┌───────┴────────┐   ┌──────────────┐
│    sales     │   │product_supplier│   │   reviews    │
│──────────────│   │────────────────│   │──────────────│
│ sale_id      │   │ product_code   │   │ review_id    │
│ product_code─┼──▶│ supplier_id    │◀──┼─product_code │
│ quantity     │   └────────────────┘   │ rating       │
│ unit_price   │           │            │ review_text  │
│ region       │   ┌───────┴────────┐   └──────────────┘
└──────────────┘   │   products     │
                   │────────────────│
                   │ product_code   │
                   │ product_name   │
                   │ brand          │
                   │ nutriscore     │
                   │ energy_100g    │
                   │ sugars_100g    │
                   └────────────────┘
```

**Design decisions:**
- **Flattened nutriments** — Nested MongoDB `nutriments` object flattened to top-level columns for direct SQL querying
- **Star schema** — Products as central dimension, sales/reviews as fact tables, all joined on `product_code`
- **Supplier extension** — Added suppliers dimension via bridge table to enable supply chain analysis

---

## Analytical Queries

### Core Queries
1. **Revenue by NutriScore** — Total revenue per nutritional grade, showing whether healthier products sell more
2. **Data quality gaps** — Products with sales but missing nutritional data, identifying governance issues
3. **Sugar distribution by grade** — Validates NutriScore system against actual nutritional content
4. **Regional revenue breakdown** — Geographic performance analysis with average order values

### Extension Queries
5. **Revenue by supplier** — Traces sales back to supplier for supply chain performance analysis
6. **Average rating by supplier** — Links customer satisfaction to supplier quality

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Raw data store | MongoDB | Semi-structured product data storage |
| Processing engine | Apache Spark (PySpark) | ETL: extract, clean, flatten nested JSON |
| File format | Parquet | Columnar storage for efficient analytics |
| Analytics database | DuckDB | In-process SQL analytics engine |
| Language | Python | Orchestration and data manipulation |

---

## Project Structure

```
food-analytics-pipeline/
├── README.md
├── notebook.ipynb              # Full pipeline notebook
├── requirements.txt
├── .gitignore
├── schema/
│   └── star_schema.sql         # DDL for the analytics schema
└── images/
    └── architecture.png
```

---

## How to Run

### Prerequisites
- Python 3.9+
- MongoDB running locally on port 27017
- Java 8+ (required for Spark)

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/food-analytics-pipeline.git
cd food-analytics-pipeline

# Install dependencies
pip install -r requirements.txt

# Import data into MongoDB (see notebook for download links)
mongoimport --db off_small_subset --collection products --file products_fallback.jsonl --drop

# Open the notebook
jupyter notebook notebook.ipynb
```

---

## Key Insights

- Products with NutriScore grade **A** generated meaningful revenue despite lower volume, suggesting a premium pricing opportunity for healthier products
- Several high-revenue products had **missing nutritional data**, flagging a data governance issue
- Sugar content validates NutriScore grading — grade A products average significantly lower sugar than grade E
- Supplier performance analysis revealed uneven quality ratings, enabling targeted supplier management

---

## Author

**Morvica Purohit** — MSc Business Analytics, UCL  
(https://www.linkedin.com/in/morvica-purohit/) · [Email](mailto:morvicapurohit2000@gmail.com)
