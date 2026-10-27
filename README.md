# Northwind sales dashboard (Power BI)

**Status:** Power BI sales dashboard model and data on public Northwind data. Not production or company data.

## Data source
Data sourced from the public Northwind OData service. Northwind is Microsoft's public dataset for a made-up food trading company (orders 1996-07-04 to 1998-05-06), read from: https://services.odata.org/V4/Northwind/Northwind.svc/ (Anonymous access). The CSVs in `data/` hold the same tables as pulled from that service on 27 Oct 2026, so the model can be inspected or rebuilt without the web feed.

## Report page
Four cards (Total Revenue, Orders, Avg Order Value, Units Sold); a monthly line of Total Revenue; bar charts of Total Revenue by Category and by Country; slicers for Year and Category.

## Tables (row counts from the pull)
| Table | Rows | Columns |
|---|---|---|
| sales (fact: one row per order line) | 2,155 | OrderID, OrderDate, CustomerID, ProductID, UnitPrice, Quantity, Discount |
| products | 77 | ProductID, ProductName, Category |
| customers | 91 | CustomerID, CompanyName, Country, City |
| calendar (DAX table in Power BI) | 1,096 | Date, Year, Month Number, Year Month |

Source entities: Order_Details (2,155), Orders (830), Customers (91), Products (77), Categories (8). Sales = Order_Details joined to Orders on OrderID. Products = Products joined to Categories (CategoryName renamed Category).

## Model
Star schema, many-to-one, single direction (dimension filters fact): Sales[ProductID] -> Products[ProductID]; Sales[CustomerID] -> Customers[CustomerID]; Sales[OrderDate] -> Calendar[Date].

## Measures (DAX, exact from the model; also in measures.dax)
```
Total Revenue = SUMX(Sales, Sales[UnitPrice] * Sales[Quantity] * (1 - Sales[Discount]))
Orders = DISTINCTCOUNT(Sales[OrderID])
Units Sold = SUM(Sales[Quantity])
Avg Order Value = DIVIDE([Total Revenue], [Orders])
Revenue Prior Year = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Calendar'[Date]))
Revenue YoY % = DIVIDE([Total Revenue] - [Revenue Prior Year], [Revenue Prior Year])
Avg Discount % = DIVIDE(SUMX(Sales, Sales[UnitPrice] * Sales[Quantity] * Sales[Discount]), SUMX(Sales, Sales[UnitPrice] * Sales[Quantity]))
```
The report page shows Total Revenue, Orders, Avg Order Value and Units Sold as cards. Revenue Prior Year, Revenue YoY % and Avg Discount % are in the model but not on the page.

## Checked numbers (calculated from the same pull)
Total Revenue 1,265,793.04; Orders 830; Units 51,317; Avg Order Value 1,525.05. Revenue by year: 1996 208,083.97; 1997 617,085.20; 1998 440,623.87.
The 1998-05 point on the monthly line is a partial month (last order 1998-05-06), not a sales drop.

## Validation status
Opened and refreshed in Power BI Desktop with the cards and charts loading; Year + Category slicers filtered the cards and charts as expected (1996 + Confections: 56 orders, 1,357 units, revenue 29,685.55, matching the calculation above). Not tested: Service publishing, refresh schedules, row-level security.
