# 🗽 NYC Data Fusion & Analytical Database Platform

An end-to-end data engineering and exploratory analytics project integrating multiple NYC public datasets into a unified, high-performance PostgreSQL star schema for multi-dimensional urban analysis.

This project combines:

- 📞 NYC 311 Service Requests  
- 🏘 NYC Property Sales  
- 📊 NYC Demographics Data  

into a normalized relational model designed for scalable analytics and correlation analysis across socio-economic, infrastructure, and real estate dimensions.

---

## 📌 Project Overview

New York City produces massive volumes of public data. Individually, these datasets are useful. Together, they become powerful.

The objective of this project was to:

- Integrate heterogeneous public datasets into a unified relational schema
- Enforce data integrity using normalization and constraints
- Optimize for analytical querying using a Star Schema design
- Enable advanced correlation analysis between:
  - Property valuation
  - Quality-of-life complaints (311)
  - Socio-economic indicators

The final system is deployed on a **Neon Serverless PostgreSQL instance**.

---

## 🏗 Architecture & Design

### Schema Type: Star Schema (3NF)

The database follows a simplified Star Schema model:

- **1 Central Dimension Table**
- **2 Fact Tables**
- Analytical Views
- Stored Functions
- Business Rule Trigger

This design enables fast joins and scalable analytical querying.

---

## 🗂 Database Schema

### 1️⃣ `dim_nyc_demographics` (Dimension Table – 3NF)

Primary geographical and socio-economic lookup table.

| Column | Type | Constraints |
|--------|------|-------------|
| zip_code | TEXT | Primary Key |
| total_population | INTEGER | CHECK ≥ 0 |
| median_income | NUMERIC | CHECK ≥ 0 |
| borough | TEXT | NOT NULL |

---

### 2️⃣ `fact_311_requests` (Fact Table)

Stores granular service request events.

| Column | Type | Constraints |
|--------|------|-------------|
| unique_key | TEXT | Primary Key |
| created_date | TIMESTAMP | NOT NULL, Indexed |
| complaint_type | TEXT | Indexed |
| incident_zip | TEXT | FK → dim_nyc_demographics.zip_code |
| status | TEXT | NOT NULL |

**Foreign Key Behavior:**  
`ON DELETE SET NULL` to preserve historical records.

---

### 3️⃣ `fact_property_sales` (Fact Table)

Stores individual property sales transactions.

| Column | Type | Constraints |
|--------|------|-------------|
| sale_id | SERIAL | Primary Key |
| neighborhood | TEXT |  |
| building_class_category | TEXT |  |
| zip_code | TEXT | FK → dim_nyc_demographics.zip_code |
| sale_price | NUMERIC | NOT NULL, CHECK ≥ 0, Indexed |
| sale_date | DATE | NOT NULL |

---

## ⚙️ Logic Layer

This project goes beyond static storage and includes a logic layer.

### 📊 Views

#### `view_zip_wealth_summary`
- Aggregates total sales and average sale price per zip code
- Linked to borough and median income
- Used to identify market trends and undervalued areas

#### `view_high_complaint_areas`
- Aggregates 311 complaint volume by zip code
- Used to detect infrastructure stress points

---

### 🧠 Stored Function

#### `get_market_rating(input_zip TEXT)`

Classifies zip codes into:

- `Prime Market` (Avg price > $1,000,000)
- `Standard Market` ($500,000 – $1,000,000)
- `Budget Market` (< $500,000)

Enables quick geographical segmentation.

---

### 🔁 Trigger

#### `trg_auto_luxury`

A `BEFORE INSERT` trigger on `fact_property_sales`.

If:

```sql
sale_price > 5,000,000
