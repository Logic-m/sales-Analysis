# sales-Analysis

# 📊 Sales Performance Dashboard

An interactive Power BI dashboard analyzing sales, profitability, and regional performance from a 1,000-record retail sales dataset spanning January 2023 – January 2024.

 <img src="Screenshot 2026-08-29 142855.png" alt="Project 2 Placeholder" width="100%"/>

---

## 📁 Project Overview

This project takes raw transactional sales data and turns it into a decision-ready dashboard covering:

- Overall sales and profit performance
- Regional and sales rep comparisons
- Product category breakdowns
- Monthly sales and profit trends
- Channel and customer-type behavior

The underlying data was cleaned and feature-engineered in Python (pandas) before being loaded into Power BI for modeling and visualization.

---

## 🗃️ Dataset

**File:** `cleaned_sales_data.csv`
**Rows:** 1,000 sales transactions
**Date range:** 2023-01-01 to 2024-01-01

### Original columns
| Column | Description |
|---|---|
| `Product_ID` | Unique identifier for the product sold |
| `Sale_Date` | Date the sale occurred |
| `Sales_Rep` | Sales representative who closed the sale (Alice, Bob, Charlie, David, Eve) |
| `Region` | Sales region (North, South, East, West) |
| `Sales_Amount` | Total revenue recorded for the transaction |
| `Quantity_Sold` | Number of units sold |
| `Product_Category` | Category of the product (Furniture, Food, Clothing, Electronics) |
| `Unit_Cost` | Cost per unit |
| `Unit_Price` | Sale price per unit |
| `Customer_Type` | New or Returning customer |
| `Discount` | Discount rate applied (0–1) |
| `Payment_Method` | Cash, Credit Card, or Bank Transfer |
| `Sales_Channel` | Online or Retail |

### Engineered columns
| Column | Description | Formula |
|---|---|---|
| `Year` | Year extracted from `Sale_Date` | `YEAR(Sale_Date)` |
| `Month` | Month name extracted from `Sale_Date` | `FORMAT(Sale_Date, "MMMM")` |
| `Weekday` | Day of week extracted from `Sale_Date` | `Sale_Date.dt.day_name()` |
| `Profit` | Gross profit per transaction | `(Unit_Price - Unit_Cost) * Quantity_Sold` |
| `Profit_Margin_Percent` | Profit as a % of sales amount | `Profit / Sales_Amount * 100` |
| `Discount_Amount` | Dollar value of the discount given | `Quantity_Sold * Unit_Price * Discount` |
| `Sales_Performance_Category` | Tercile bucket of transaction size | `qcut(Sales_Amount, 3, [Low, Medium, High])` |

### ⚠️ Known Data Quality Note
621 of the 1,000 rows currently have a missing `Sale_Date` (and therefore missing `Year`/`Month`). These rows are **intentionally retained** rather than dropped, since the other fields (sales amount, profit, region, rep, etc.) are still valid and valuable. See the [Data Cleaning](#-data-cleaning--fixes) section below for the recommended fix.

---

## 📈 Key Metrics (Current Snapshot)

| KPI | Value |
|---|---|
| Total Sales | $5.02M |
| Total Profit | $6.49M |
| Total Orders | 1,000 |
| Overall Profit Margin | ~129% *(see note below)* |

> **Note:** The profit margin exceeds 100% because `Profit` is calculated from `Unit_Price`/`Unit_Cost`/`Quantity_Sold`, while `Sales_Amount` is a separately recorded field that doesn't always equal `Quantity_Sold × Unit_Price`. This is a known inconsistency in the source data — see Known Limitations below.

---

## 🖥️ Dashboard Pages & Visuals

### Page 1 — Sales Overview
- **KPI Cards:** Total Sales, Total Profit, Total Orders, Overall Profit Margin %
- **Total Profit by Region** — bar chart comparing North, West, East, South
- **Total Sales and Overall Profit Margin % by Sales Rep** — combo bar chart across Alice, Bob, Charlie, David, Eve
- **Total Profit and Total Sales by Month** — line chart trend across the year
- **Total Sales by Product Category** — donut chart (Furniture, Electronics, Food, Clothing)

### Sidebar Filters
- **Year** — 2023 / 2024
- **Region** — North, South, East, West
- **Sales_Channel** — Online / Retail (button slicer)
- **Product_Category** — Clothing, Electronics, Food, Furniture

---

## 🧹 Data Cleaning & Fixes

The dataset went through the following cleaning steps before/within Power BI:

1. **Missing value & duplicate check** — verified in pandas; no duplicate rows found.
2. **Date parsing** — `Sale_Date` converted to a proper date type; rows where parsing failed were **kept**, not dropped.
3. **Recommended fix for missing `Sale_Date`/`Year`/`Month`** (in Power Query):
   - Sort by `Sales_Rep`, then `Sale_Date`
   - Right-click `Sale_Date` → **Fill Down** to carry forward the nearest known date within each rep's records
   - Regenerate `Year` (`Add Column → Date → Year`) and `Month` (`Add Column → Date → Month → Name of Month`) from the corrected dates
   - Set `Month`'s **Sort by Column** property to a numeric `Month_Num` column so charts display Jan–Dec in order, not alphabetically
4. **Outlier check** — IQR method applied to `Sales_Amount`; no outliers found outside the boundaries.
5. **Feature engineering** — `Profit`, `Profit_Margin_Percent`, `Discount_Amount`, and `Sales_Performance_Category` added as described above.

---

## 🛠️ Tech Stack

- **Python (pandas, numpy, matplotlib, seaborn)** — initial data cleaning, exploration, and feature engineering (see `Sales_Analysis.ipynb`)
- **Power BI Desktop** — data modeling, DAX measures, and interactive dashboard
- **Power Query (M)** — date correction and column derivation

---

## 📐 Core DAX Measures

```DAX
Total Sales = SUM(cleaned_sales_data[Sales_Amount])
Total Profit = SUM(cleaned_sales_data[Profit])
Total Orders = COUNTROWS(cleaned_sales_data)
Overall Profit Margin % = DIVIDE([Total Profit], [Total Sales])
Avg Order Value = DIVIDE([Total Sales], [Total Orders])
```

---

## 📂 Repository Structure

```
├── README.md                     # This file
├── cleaned_sales_data.csv        # Cleaned + feature-engineered dataset
├── Sales_Analysis.ipynb          # Python data cleaning & EDA notebook
├── PowerBI_Dashboard_Guide.md    # Full Power BI build guide (data model, DAX, layout)
└── dashboard_preview.png         # Dashboard screenshot
```

---

## 🚀 How to Reproduce

1. Open `Sales_Analysis.ipynb` to review or re-run the cleaning and feature engineering steps in Python.
2. Open Power BI Desktop → **Get Data → Text/CSV** → load `cleaned_sales_data.csv`.
3. Follow `PowerBI_Dashboard_Guide.md` for the full data model, DAX measures, and page layout.
4. Apply the `Sale_Date` fill-down fix described above before building any month/year-based visuals.
5. Build the KPI cards, charts, and slicers as shown in the dashboard preview.

---

## 🔍 Key Insights

- Total profit is fairly evenly distributed across all four regions, with North and West leading slightly.
- David and Bob are the top-performing sales reps by total sales among the five reps.
- Sales spiked sharply in January before flattening out for the rest of the year — worth investigating whether this reflects a seasonal pattern or a data entry effect.
- Product category sales are split across Furniture, Electronics, Food, and Clothing with no single category dominating.

---

## 📌 Future Improvements

- Resolve the `Sale_Date` gaps at the source so all 1,000 transactions have complete time-based reporting.
- Reconcile the `Sales_Amount` vs. `Quantity_Sold × Unit_Price` discrepancy to get an accurate (sub-100%) profit margin.
- Add year-over-year and month-over-month growth measures once a full year of clean date data is available.
- Add drill-through pages for individual sales reps and regions.
