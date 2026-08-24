# Telecom Customer Churn Analysis — Power BI Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-00A4EF?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
> [!IMPORTANT]
> This project analyzes a dataset of **1,000,000 customer records** (`customer_churn_1M`) — cleaned, transformed, and modeled end-to-end using SQL, Power Query, and DAX.

An end-to-end analytics project that cleans, models, and visualizes customer churn for a telecom company — built to identify **who** is churning, **why**, and **how much revenue** is at stake, using SQL, Power Query, and Power BI.

---
## Dashboard Preview

🔗 [Click here to view the full interactive dashboard](https://drive.google.com/file/d/1YrCYW5VCL318LhkAUd05AGgJXT-TyLbb/view?usp=sharing)


## Table of Contents

- [Business Problem](#business-problem)
- [Dataset](#dataset)
- [Tools & Technologies](#tools--technologies)
- [Project Workflow](#project-workflow)
- [SQL — Data Cleaning & Exploration](#sql--data-cleaning--exploration)
- [Power Query — Transformation Steps](#power-query--transformation-steps)
- [DAX Measures](#dax-measures)
- [Dashboard Highlights](#dashboard-highlights)
- [Key Insights](#key-insights)
- [Project Structure](#project-structure)
- [How to Use](#how-to-use)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## Business Problem

Customer churn is one of the costliest problems in telecom — acquiring a new customer typically costs 5–7x more than retaining an existing one. This project analyzes a customer-level dataset to answer:

- What is the overall churn rate, and how does it vary by customer segment?
- Which contract types, payment methods, or tenure bands have the highest churn?
- How much monthly and total revenue is lost to churn?
- Which demographic groups (senior citizens, customers with partners/dependents) churn more or less?
- What early signals (short tenure, month-to-month contracts) predict churn risk?

## Dataset

| Property | Detail |
|---|---|
| **Table name** | `customer_churn_1M` |
| **Size** | ~1,000,000 customer records |
| **Grain** | One row per customer |

**Key columns**

| Column | Description |
|---|---|
| `customerID` | Unique customer identifier |
| `gender` | Male / Female |
| `SeniorCitizen` | 1 = senior citizen, 0 = not |
| `Partner` | Whether the customer has a partner (Yes/No) |
| `Dependents` | Whether the customer has dependents (Yes/No) |
| `tenure` | Number of months the customer has stayed |
| `Contract` | Month-to-month, One year, Two year |
| `PaymentMethod` | How the customer pays |
| `MonthlyCharges` | Current monthly bill |
| `TotalCharges` | Total amount billed to date |
| `Churn` | Yes/No — whether the customer left |

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **SQL** | Initial data exploration, quality checks, and cleaning at the source/database layer |
| **Power Query (M)** | In-Power BI data transformation, type correction, and derived columns |
| **DAX** | Calculated measures for churn rate, revenue, and segment KPIs |
| **Power BI Desktop** | Data modeling and interactive dashboard/report design |
| **Power BI Service** *(optional)* | Publishing and sharing the report |
| **Git / GitHub** | Version control and project documentation |

## Project Workflow

```
Raw Data (CSV/Excel/SQL source)
        │
        ▼
   SQL Cleaning        →  data_cleaning.sql
        │
        ▼
 Power Query Transform  →  power_query_transform.pq
        │
        ▼
  Power BI Data Model    (relationships, calculated columns)
        │
        ▼
   DAX Measures          →  dax_measures.pdf
        │
        ▼
  Interactive Dashboard  →  dashboard.pbix
```

## SQL — Data Cleaning & Exploration

Full script: [`Customer churn SQL Quiries.pdf`](Customer%20churn%20SQL%20Quiries.pdf). Key steps:

```sql
-- Check for duplicate customer records
SELECT customerID, COUNT(*) AS occurrences
FROM customer_churn_1M
GROUP BY customerID
HAVING COUNT(*) > 1;

-- Identify blank TotalCharges (new customers with tenure = 0)
SELECT customerID, tenure, TotalCharges
FROM customer_churn_1M
WHERE TRIM(TotalCharges) = '' OR TotalCharges IS NULL;

-- Fix TotalCharges: replace blanks with 0, cast to numeric
UPDATE customer_churn_1M
SET TotalCharges = 0
WHERE TRIM(TotalCharges) = '';

ALTER TABLE customer_churn_1M
ALTER COLUMN TotalCharges DECIMAL(10,2);

-- Add TenureGroup for grouped analysis
ALTER TABLE customer_churn_1M
ADD TenureGroup VARCHAR(10);

UPDATE customer_churn_1M
SET TenureGroup =
    CASE
        WHEN tenure <= 12 THEN '0-1 yr'
        WHEN tenure <= 24 THEN '1-2 yr'
        WHEN tenure <= 48 THEN '2-4 yr'
        WHEN tenure <= 60 THEN '4-5 yr'
        ELSE '5+ yr'
    END;
```

The complete script also covers duplicate removal, NULL audits across all key columns, text standardization, and a final post-cleaning sanity check — see the full file for all 13 steps.

## Power Query — Transformation Steps

Full script: [`power_query_transform.pq`](power_query_transform.pq). Applied steps, in order:

1. Load source data
2. Promote headers
3. Remove duplicate rows
4. Trim whitespace from text columns
5. Replace blank `TotalCharges` with `0`
6. Correct data types (`SeniorCitizen`, `tenure` → whole number; `MonthlyCharges`, `TotalCharges` → decimal)
7. Add `SeniorCitizenLabel` (0/1 → No/Yes) for readable visuals
8. Add `TenureGroup` for grouped churn analysis
9. Remove rows with missing `customerID`

## DAX Measures

Full reference: [`dax_measures.pdf`](dax_measures.pdf).

**Core**
```dax
Total Customers = COUNTROWS(customer_churn_1M)

Total Churned Customers =
CALCULATE(COUNTROWS(customer_churn_1M), customer_churn_1M[Churn] = "Yes")

Churn Rate % = DIVIDE([Total Churned Customers], [Total Customers], 0)

Retention Rate % = 1 - [Churn Rate %]
```

**Demographics**
```dax
Senior Citizen % =
DIVIDE(
    CALCULATE(COUNTROWS(customer_churn_1M), customer_churn_1M[SeniorCitizen] = 1),
    [Total Customers], 0
)

Partner % =
DIVIDE(
    CALCULATE(COUNTROWS(customer_churn_1M), customer_churn_1M[Partner] = "Yes"),
    [Total Customers], 0
)

Dependents % =
DIVIDE(
    CALCULATE(COUNTROWS(customer_churn_1M), customer_churn_1M[Dependents] = "Yes"),
    [Total Customers], 0
)
```

**Revenue**
```dax
Total Revenue = SUM(customer_churn_1M[TotalCharges])

ARPU = DIVIDE(SUM(customer_churn_1M[MonthlyCharges]), [Total Customers], 0)

Revenue Lost =
CALCULATE(SUM(customer_churn_1M[MonthlyCharges]), customer_churn_1M[Churn] = "Yes")

Revenue by Contract Type =
CALCULATE(SUM(customer_churn_1M[TotalCharges]), ALLEXCEPT(customer_churn_1M, customer_churn_1M[Contract]))
```

**Tenure**
```dax
Tenure Group =
SWITCH(
    TRUE(),
    customer_churn_1M[tenure] <= 12, "0-1 yr",
    customer_churn_1M[tenure] <= 24, "1-2 yr",
    customer_churn_1M[tenure] <= 48, "2-4 yr",
    customer_churn_1M[tenure] <= 60, "4-5 yr",
    "5+ yr"
)

Churn Rate by Tenure Group =
DIVIDE(
    CALCULATE(COUNTROWS(customer_churn_1M), customer_churn_1M[Churn] = "Yes"),
    CALCULATE(COUNTROWS(customer_churn_1M)),
    0
)
```

## Dashboard Highlights

- **KPI cards:** Total Customers, Churn Rate %, Revenue Lost, ARPU
- **Churn by Contract Type** — bar chart comparing month-to-month vs. annual contracts
- **Churn by Tenure Group** — highlights early-tenure churn risk
- **Demographic breakdown** — Senior Citizen %, Partner %, Dependents %
- **Revenue impact** — revenue lost to churned customers vs. total revenue

## Key Insights

*(Replace with your actual findings once the dashboard is finalized — e.g. "Month-to-month contract customers churn at 3x the rate of two-year contract customers" or "42% of churned revenue comes from customers with tenure under 12 months.")*

## Project Structure

```
├── README.md
├── data_cleaning.sql              # SQL scripts for cleaning & transforming raw data
├── power_query_transform.pq       # Power Query (M) transformation steps
├── dax_measures.pdf               # All DAX measures used in the data model
├── dashboard.pbix                 # Power BI dashboard file
└── screenshots/                   # Dashboard preview images
```

## How to Use

1. Clone this repository.
2. Review `data_cleaning.sql` to see the source-level cleaning logic (or run it against your own database copy).
3. Open `dashboard.pbix` in Power BI Desktop — the Power Query steps in `power_query_transform.pq` are already applied inside the file.
4. Refresh the data source if pointing to your own copy of `customer_churn_1M`.
5. Explore the report pages and slicers to filter by contract type, tenure group, or demographics.

## Future Improvements

- Add a predictive churn-risk score (logistic regression or Power BI's built-in AI visuals)
- Incorporate customer support/ticket data for a fuller churn-driver picture
- Automate data refresh via a scheduled pipeline

## Author

**Laxman Ram**
Data Analytics Enthusiast | Business Intelligence, KPI Dashboarding & Data Visualization
B.Tech, National Institute of Technology (NIT), Raipur

- Email: lr7797901@gmail.com
