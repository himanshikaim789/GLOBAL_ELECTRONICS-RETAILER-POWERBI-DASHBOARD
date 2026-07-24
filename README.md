# GLOBAL_ELECTRONICS-RETAILER-POWERBI-DASHBOARD
# 🌐 Global Electronics Retailer — Power BI Dashboard

An end-to-end data analytics project: raw CSVs → cleaned data model → star schema → DAX measures → a single-page Power BI dashboard.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white)

---

## 📌 Project Overview

This project takes the **Global Electronics Retailer** dataset — sales transactions across 8 countries and 5 years — and turns it into a fully modeled, interactive Power BI dashboard. The focus wasn't just building visuals; it was getting the data *right* before the report canvas ever opens: fixing encoding issues, correcting data types, resolving a multi-currency relationship problem, and handling nulls honestly instead of hiding them.

**[Add your dashboard screenshot here once published]**

---

## 🗂️ Dataset

| Table | Rows | Description |
|---|---|---|
| Sales | 62,884 | Order-level transactions, Jan 2016 – Feb 2021 |
| Customers | 15,266 | Customer demographics across 8 countries |
| Products | 2,517 | Product catalog across 8 categories, 11 brands |
| Stores | 67 | Physical + 1 online store across 8 countries |
| Exchange Rates | 11,215 | Daily FX rates for USD, EUR, GBP, CAD, AUD |

Source data required cleanup before modeling:
- Fixed a Latin-1 encoding issue in `Customers.csv` (non-UTF8 characters in names)
- Converted `Unit Cost/Price USD` from currency-formatted strings (`"$6.62 "`) to numeric
- Preserved `Zip Code` as text to keep leading zeros intact
- Correctly typed 79% of blank `Delivery Date` values as nulls rather than treating them as errors

---

## 🏗️ Data Model

Built as a star schema with **Sales** as the fact table:

```
        Customers
             \
Products — Sales — Stores
             /
   Exchange Rates (via merged Currency+Date key)
             |
          Date (dedicated calendar table)
```

Relationships:
- `Sales[CustomerKey] → Customers[CustomerKey]` (many-to-one)
- `Sales[StoreKey] → Stores[StoreKey]` (many-to-one)
- `Sales[ProductKey] → Products[ProductKey]` (many-to-one)
- `Sales[Order Date] → Date[Date]` (many-to-one, marked date table)
- `Sales[CurrencyDateKey] → Exchange Rates[CurrencyDateKey]` — a composite key built to solve the classic problem of relating on two columns (Currency + Date) at once, since Power BI relationships only support a single column

---

## 📐 Key DAX Measures

```dax
Revenue = SUMX(Sales, Sales[Quantity] * RELATED(Products[Unit Price USD]))
Profit = [Revenue] - SUMX(Sales, Sales[Quantity] * RELATED(Products[Unit Cost USD]))
Profit Margin % = DIVIDE([Profit], [Revenue])
Revenue YoY % = DIVIDE([Revenue] - CALCULATE([Revenue], SAMEPERIODLASTYEAR('Date'[Date])), 
                        CALCULATE([Revenue], SAMEPERIODLASTYEAR('Date'[Date])))
Revenue per Sqm = DIVIDE([Revenue], SUM(Stores[Square Meters]))
```

Full measure list is in the `/dax` folder.

---

## 📊 Dashboard

One consolidated page, filterable by Year and Category:
- KPI cards — Revenue, Profit, Profit Margin %, Total Orders
- Revenue trend over time (with YoY comparison)
- Revenue by product category
- Revenue by customer country (map)
- Revenue by store country

---

## 🔍 Notable Findings

- **Delivery data has a major coverage gap** — 79% of orders show no delivery date, which needs to be flagged (not silently excluded) before building any fulfillment-rate KPI on top of it.
- **The Online store has no physical footprint**, correctly modeled as a null rather than a 0 to avoid skewing revenue-per-square-meter calculations.
- **Multi-currency sales** required a composite relationship key rather than a simple lookup, since Power BI can't relate on two columns natively.

---

## 🛠️ Tools Used

- **Python (pandas, openpyxl)** — data cleaning and type correction
- **Power Query** — transformation and composite key creation
- **Power BI Desktop** — data modeling, DAX, dashboard build
- **DAX** — time intelligence, profitability, and efficiency measures

---

## 📁 Repository Structure

```
├── data/
│   ├── Sales.csv
│   ├── Customers.csv
│   ├── Products.csv
│   ├── Stores.csv
│   └── Exchange_Rates.csv
├── dax/
│   └── measures.dax
├── Global_Electronics_Retailer_Dashboard.pbix
└── README.md
```

---

## 📬 Connect

If you spot ways to improve the model or have handled multi-currency/partial-delivery data differently, I'd love to hear your approach — feel free to open an issue or connect on LinkedIn.
