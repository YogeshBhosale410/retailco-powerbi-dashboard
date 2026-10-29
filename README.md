# RetailCo Sales Performance Dashboard (Power BI)

An end-to-end Power BI practice project: cleaning a messy multi-table dataset in Power Query, building a star-schema data model, writing DAX measures, and designing a one-page interactive sales dashboard.

> **Note:** This project uses a **synthetic (dummy) dataset** created for practice. The figures do not represent a real company.

---

## Dashboard preview

![Dashboard (https://github.com/YogeshBhosale410/retailco-powerbi-dashboard/blob/main/dashbord.png")

---

## Business questions the dashboard answers

- How much did RetailCo sell, and how much profit did it make?
- Is performance improving compared with last year?
- Which states, categories and products drive the most sales?
- What share of orders are delivered, returned or cancelled?
- How do results change by year and by region?

---

## Dataset

One Excel workbook (`RetailCo_Dummy_Data.xlsx`) with four sheets:

| Sheet | Rows | Description |
|---|---|---|
| **Sales** (fact) | 2,015 raw (2,000 unique orders) | OrderID, OrderDate, ShipDate, CustomerID, ProductID, Quantity, Discount, PaymentMode, OrderStatus |
| **Customers** | 200 | CustomerID, CustomerName, City, State, Region, Segment, Age, JoinDate |
| **Products** | 26 | ProductID, ProductName, Category, SubCategory, UnitPrice, CostPrice |
| **RegionManagers** | 4 | Region, RegionManager, Email |

Period covered: **January 2023 to December 2025**.

The raw data was made messy on purpose, to practise cleaning:

- 15 duplicate order rows
- Blank values in `Discount`, `PaymentMode` and `Age`
- Inconsistent text case in `PaymentMode`
- Extra spaces and uppercase in some `CustomerName` values
- `OrderDate` stored as text

---

## Tools used

- **Power BI Desktop**: Power Query, data modelling, DAX, report design
- **Excel**: source data

---

## 1. Data cleaning (Power Query)

- Removed duplicate rows on `OrderID` (2,015 rows reduced to **2,000**)
- Converted `OrderDate` from text to date
- Replaced blank `Discount` with 0
- Replaced blank `PaymentMode` with "Unknown" and standardised the text case
- Trimmed and capitalised `CustomerName`
- Replaced blank `Age` with the average age
- Added calculated columns such as `DeliveryDays` (ShipDate minus OrderDate) and `MarginUnit` (UnitPrice minus CostPrice)
- Renamed queries to clear table names

---

## 2. Data model

A star schema with `Sales_fact` at the centre:

```
Customers_dim ──┐
                ├──► Sales_fact ◄── DIM-DATE
Products ───────┘
```

| Relationship | Type |
|---|---|
| `Customers_dim[CustomerID]` → `Sales_fact[CustomerID]` | One-to-many, single direction |
| `Products[ProductID]` → `Sales_fact[ProductID]` | One-to-many, single direction |
| `DIM-DATE[Date]` → `Sales_fact[OrderDate]` | One-to-many, single direction, active |
| `DIM-DATE[Date]` → `Sales_fact[ShipDate]` | Inactive |

- `DIM-DATE` is a calendar table created with DAX and marked as the date table.
- `Month` is sorted by `MonthNo`.
- Key columns are hidden in report view.
- Hierarchies: Category > SubCategory > ProductName, and Region > State > City.

---

## 3. DAX measures

```dax
Total Sales = SUMX(Sales_fact, Sales_fact[Quantity] * RELATED(Products[UnitPrice]) * (1 - Sales_fact[Discount]))
Total Cost = SUMX(Sales_fact, Sales_fact[Quantity] * RELATED(Products[CostPrice]))
Total Profit = [Total Sales] - [Total Cost]
Profit Margin % = DIVIDE([Total Profit], [Total Sales])
Total Orders = DISTINCTCOUNT(Sales_fact[OrderID])
Avg Order Value = DIVIDE([Total Sales], [Total Orders])

Sales LY = CALCULATE([Total Sales], DATEADD('DIM-DATE'[Date], -1, YEAR), ALL('DIM-DATE'[Year]))
Profit LY = CALCULATE([Total Profit], DATEADD('DIM-DATE'[Date], -1, YEAR), ALL('DIM-DATE'[Year]))
Orders LY = CALCULATE([Total Orders], DATEADD('DIM-DATE'[Date], -1, YEAR), ALL('DIM-DATE'[Year]))
Margin LY = DIVIDE([Profit LY], [Sales LY])

YoY Growth % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])
YoY Label =
VAR g = [YoY Growth %]
RETURN IF(ISBLANK(g), "No prior year", FORMAT(g, "+0.0%;-0.0%") & " vs LY")
YoY Color = IF([YoY Growth %] >= 0, "#2DD4BF", "#F97316")
```

---

## 4. Dashboard features

- **Slicers:** Year and Region (tile style, with a highlighted selected state)
- **KPI cards:** Total Sales, Total Profit, Profit Margin, Total Orders, Average Order Value
- **Monthly trend:** Total Sales vs Sales LY line chart
- **Sales by State:** bar chart
- **Top 5 Products:** bar chart with a Top N filter
- **Sales by Category:** bar chart
- **Order Status:** donut chart (Delivered, Returned, Cancelled)
- Dark theme, consistent card styling, aligned layout

---

## 5. Key figures (all years, no filters)

| Metric | Value |
|---|---|
| Total Sales | about ₹58.3M |
| Total Profit | about ₹12.4M |
| Profit Margin | about 21% |
| Total Orders | 2,000 |
| Average Order Value | about ₹29,000 |

Other observations from the dashboard:

- **Electronics** is the largest category by far, followed by Furniture.
- The top 5 products are Lenovo IdeaPad, HP 15s, Asus Vivobook, Galaxy M and Redmi Note.
- About two-thirds of sales value comes from delivered orders; the rest is returned or cancelled.

---

## 6. Skills demonstrated

- Data cleaning and transformation in Power Query
- Star-schema modelling, relationships and a DAX date table
- DAX: `SUMX`, `RELATED`, `CALCULATE`, `DIVIDE`, `DISTINCTCOUNT`, `DATEADD`, `ALL`, `FORMAT`
- Time-intelligence comparison (year over year)
- Dashboard design, slicers, conditional colours and visual formatting

---

## 7. Limitations and possible next steps

- The state-level map visual needs a Power BI sign-in, so a bar chart is used instead.
- 2023 has no "last year" comparison because it is the first year in the data.
- Planned improvements: a metric switcher (Sales / Profit / Orders), a custom tooltip page, a clear-filters button, and row-level security by region.

---

## Repository structure

```
├── README.md
├── RetailCo_Dummy_Data.xlsx
├── RetailCo_Sales_Dashboard.pbix
└── images/
    └── dashboard.png
```

---

## How to open

1. Download `RetailCo_Sales_Dashboard.pbix` and open it in **Power BI Desktop**.
2. If prompted, update the Excel file path: **Home > Transform data > Data source settings**, then point to `RetailCo_Dummy_Data.xlsx`.
3. Click **Refresh**.

---

## Author

**Yogesh Bhosale**
BCA graduate pursuing a Data Analyst role.
LinkedIn: linkedin.com/in/yogesh-bhosale-71391231a | GitHub: [_add your link_](https://github.com/YogeshBhosale410/retailco-powerbi-dashboard)
