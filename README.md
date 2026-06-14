# Northwind Foods — SQL Business Analysis

Post-merger data analysis | PostgreSQL | Business Intelligence

---

This is a training project built around the Northwind Foods dataset — a global B2B company that imports and exports gourmet products, sells to corporate clients, and ships via freight carriers. The scenario: a post-merger integration where the goal is to establish an operational baseline using SQL.

The analysis covers four business domains: inventory and capital, supplier networks, logistics performance, and geographic market intelligence.

---

## Technical Stack

| Layer | Detail |
|---|---|
| Database | PostgreSQL / MySQL |
| Concepts | Multi-table JOINs, CTEs, Subqueries, Aggregations, Reusable Views |
| Standards | Single-query architecture, descriptive aliasing, iterative development |

---

## Database Schema

```
Products ──────┐
               ├──► Order_Details ──► Orders ──► Customers
Categories ────┘                         │
                                         └──► Shippers
Suppliers ──► Products
```

| Table | Role |
|---|---|
| Products | SKU catalog — IDs, prices, stock levels |
| Customers | B2B client profiles and locations |
| Orders | Transactions — timestamps and shipping metrics |
| Order_Details | Line-item revenue at point of sale |
| Categories | Product segment classification |
| Suppliers | Vendor origins and procurement contacts |
| Shippers | Freight carrier registry |

---

## Part 1 — Baseline Metrics

How large is the business? Three queries establish the operational foundation.

```sql
-- Total product portfolio
SELECT COUNT(ProductID) AS number_products
FROM Products;
-- 77 products

-- Corporate client base
SELECT COUNT(CustomerID) AS num_customers
FROM Customers;
-- 93 B2B clients

-- Monthly transaction volume (March 1997)
SELECT COUNT(*) AS num_orders_march
FROM Orders
WHERE YEAR(OrderDate) = 1997
  AND MONTH(OrderDate) = 3;
-- 30 orders
```

---

## Part 2 — Geographic Order Distribution

Which countries receive the most shipments?

```sql
SELECT
    COUNT(o.OrderID) AS num_orders,
    c.Country
FROM Orders AS o
JOIN Customers AS c ON o.CustomerID = c.CustomerID
GROUP BY c.Country
ORDER BY num_orders DESC;
```

| Country | Orders |
|---|---|
| Brazil | 83 |
| Austria | 40 |
| Canada | 30 |
| Belgium | 19 |
| Argentina | 16 |

Linking orders to customer geographies shows which international corridors carry the most freight volume, which directly affects carrier allocation decisions.

---

## Part 3 — Revenue Analysis (Q2/Q3 1997)

Gross revenue across a four-month window, calculated at the line-item level.

```sql
SELECT
    SUM(od.Quantity * od.UnitPrice) AS TotalRevenue
FROM Orders AS o
JOIN Order_Details AS od ON o.OrderID = od.OrderID
WHERE o.OrderDate BETWEEN '1997-04-01' AND '1997-07-31';
-- 207,076.02
```

A note on pricing: `Order_Details.UnitPrice` captures the selling price locked at the time of sale. `Products.UnitPrice` reflects the current wholesale cost from suppliers. Keeping these separate ensures historical revenue figures stay accurate regardless of future price changes.

---

## Part 4 — Inventory and Capital Exposure

Which product categories tie up the most warehouse capital?

```sql
-- Stock distribution by category
SELECT
    c.CategoryName,
    COUNT(p.ProductID)  AS num_products,
    SUM(p.UnitsInStock) AS units_in_stock
FROM Products AS p
JOIN Categories AS c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
ORDER BY units_in_stock DESC
LIMIT 3;
```

| Category | Products | Units in Stock |
|---|---|---|
| Seafood | 12 | 701 |
| Beverages | 12 | 559 |
| Condiments | 12 | 507 |

```sql
-- Monetary valuation of warehouse and incoming orders
SELECT
    c.CategoryName,
    SUM(p.UnitsInStock * p.UnitPrice)                    AS cost_warehouse,
    SUM(p.UnitsOnOrder * p.UnitPrice)                    AS cost_orders,
    SUM((p.UnitsInStock + p.UnitsOnOrder) * p.UnitPrice) AS cost_overall
FROM Products AS p
JOIN Categories AS c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
ORDER BY cost_overall DESC
LIMIT 3;
```

| Category | Warehouse | On Order | Total Exposure |
|---|---|---|---|
| Seafood | 13,010 | 1,965 | 14,975 |
| Condiments | 12,024 | 2,400 | 14,424 |
| Dairy Products | 11,271 | 2,785 | 14,056 |

Seafood carries the highest capital commitment despite having the same SKU count as the other top categories — a flag for working capital review.

---

## Part 5 — Supplier Network Analysis

A reusable View is built first to avoid repeating expensive joins across all subsequent supplier queries.

```sql
CREATE VIEW view_supplier_product_catalog AS
SELECT
    s.SupplierID,
    s.CompanyName  AS SupplierName,
    s.Country      AS SupplierCountry,
    p.ProductID,
    p.ProductName,
    p.UnitPrice,
    p.UnitsInStock,
    p.UnitsOnOrder,
    p.Discontinued,
    c.CategoryID,
    c.CategoryName
FROM Products AS p
JOIN Suppliers AS s ON p.SupplierID = s.SupplierID
JOIN Categories AS c ON p.CategoryID = c.CategoryID;
```

**Filtered category query via View:**

```sql
SELECT ProductName, CompanyName
FROM view_supplier_product_catalog
WHERE CategoryName = 'Confections'
ORDER BY ProductName ASC
LIMIT 3;
```

| Product | Supplier |
|---|---|
| Chocolade | Zaanse Snoepfabriek |
| Gumbär Gummibärchen | Heli Süßwaren GmbH & Co. KG |
| Maxilaku | Karkki Oy |

**Cross-category vendor exclusion using subquery:**

```sql
-- Suppliers active in Grains/Cereals with zero presence in Dairy Products
SELECT CompanyName, CategoryName, ProductName
FROM view_supplier_product_catalog
WHERE CategoryName = 'Grains/Cereals'
  AND CompanyName NOT IN (
      SELECT CompanyName
      FROM view_supplier_product_catalog
      WHERE CategoryName = 'Dairy Products'
  )
ORDER BY CompanyName ASC;
```

| Supplier | Category | Product |
|---|---|---|
| G'day, Mate | Grains/Cereals | Filo Mix |
| Leka Trading | Grains/Cereals | Singaporean Hokkien Fried Mee |
| Pasta Buttini s.r.l. | Grains/Cereals | Gnocchi di nonna Alice |

**Premium vendor filtering using HAVING:**

```sql
SELECT
    CompanyName,
    AVG(UnitPrice) AS average_price
FROM view_supplier_product_catalog
WHERE CategoryName IN ('Beverages', 'Condiments')
GROUP BY CompanyName
HAVING AVG(UnitPrice) >= 17
ORDER BY average_price DESC
LIMIT 3;
```

| Supplier | Avg Price |
|---|---|
| Aux joyeux ecclésiastiques | 140.75 |
| Leka Trading | 32.73 |
| Grandma Kelly's Homestead | 32.50 |

---

## Part 6 — Logistics SLA Audit

How do the three freight carriers compare on speed, volume, and freight weight?

```sql
SELECT
    s.CompanyName                                        AS shipper,
    ROUND(AVG(DATEDIFF(o.ShippedDate, o.OrderDate)), 1) AS avg_days_to_ship,
    ROUND(AVG(o.Freight), 1)                            AS avg_freight,
    ROUND(MIN(o.Freight), 1)                            AS min_freight,
    ROUND(MAX(o.Freight), 1)                            AS max_freight,
    COUNT(o.OrderID)                                    AS num_orders
FROM Orders AS o
JOIN Shippers AS s ON o.ShipVia = s.ShipperID
GROUP BY s.CompanyName
ORDER BY avg_days_to_ship ASC;
```

| Carrier | Avg Speed | Avg Freight | Max Freight | Orders |
|---|---|---|---|---|
| Federal Shipping | 7.5 days | 80.4 | 1,007.6 | 255 |
| Speedy Express | 8.6 days | 65.0 | 458.8 | 249 |
| United Package | 9.2 days | 86.6 | 890.8 | 326 |

Federal Shipping is the fastest. United Package handles the highest order volume and heaviest average loads. Neither carrier dominates on all metrics, which points toward a split-carrier approach for different shipment profiles.

---

## Part 7 — Market Matrix (CTE + CASE WHEN)

The final query combines purchasing power with shipping compliance rates across every market — built as a single CTE to keep the logic readable.

```sql
WITH order_level AS (
    SELECT
        o.OrderID,
        c.Country,
        c.CustomerID,
        CASE
            WHEN o.ShippedDate IS NULL          THEN 'no_ship'
            WHEN o.ShippedDate < o.RequiredDate THEN 'before'
            WHEN o.ShippedDate = o.RequiredDate THEN 'on'
            ELSE 'after'
        END AS ship_status,
        SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS order_revenue
    FROM Orders o
    JOIN Customers c        ON o.CustomerID = c.CustomerID
    LEFT JOIN Order_Details od ON o.OrderID = od.OrderID
    GROUP BY o.OrderID, c.Country, c.CustomerID, ship_status
)
SELECT
    Country,
    COUNT(DISTINCT CustomerID) AS num_customers,
    COUNT(DISTINCT OrderID)    AS num_orders,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'before'  THEN 1 END) / COUNT(OrderID), 1) AS pct_early,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'after'   THEN 1 END) / COUNT(OrderID), 1) AS pct_delayed,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'no_ship' THEN 1 END) / COUNT(OrderID), 1) AS pct_unshipped,
    ROUND(SUM(order_revenue), 1) AS total_revenue
FROM order_level
GROUP BY Country
ORDER BY total_revenue DESC
LIMIT 3;
```

| Country | Customers | Orders | Early | Delayed | Unshipped | Revenue |
|---|---|---|---|---|---|---|
| USA | 13 | 122 | 91.8% | 5.7% | 2.5% | 263,537 |
| Germany | 11 | 122 | 94.3% | 3.3% | 1.6% | 244,083 |
| Austria | 2 | 40 | 92.5% | 2.5% | 5.0% | 139,497 |

Austria stands out: two clients generating more than half of Germany's total revenue. The 5% unshipped rate, however, is the highest of the three markets and flags a fulfillment gap worth investigating.

---

## SQL Concepts Covered

| Concept | Used In |
|---|---|
| INNER JOIN, LEFT JOIN | Parts 2, 3, 4, 6, 7 |
| GROUP BY + Aggregations | Parts 1, 2, 3, 4, 6 |
| HAVING | Part 5 |
| Subquery (NOT IN) | Part 5 |
| CREATE VIEW | Part 5 |
| CTE (WITH clause) | Part 7 |
| CASE WHEN | Part 7 |
| DATEDIFF, ROUND | Part 6 |

---
License / Lizenz
This project utilizes the industry-standard Northwind database schema, which is open-source and distributed under the **MIT License**. 

Dieses Projekt verwendet das standardisierte Northwind-Datenbankschema, welches als Open-Source-Software unter der **MIT-Lizenz** bereitgestellt wird.
