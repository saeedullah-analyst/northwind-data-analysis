# northwind-data-analysis
SQL data analysis project for Northwind Foods post-merger business integration.
## Profile & Core Competencies / Kompetenzprofil
### English Summary
*   **Project Goal:** Analyzed the global B2B Northwind database to deliver data-driven insights on operational scale, supply chain efficiency, procurement costs, and shipping carrier SLAs.
*   **Technical Skills Applied:** Multi-table Joins (`INNER`, `LEFT`), Advanced Aggregations (`COUNT`, `SUM`, `AVG`), Group Filtering (`HAVING`), Reusable Database Views (`CREATE VIEW`), Subqueries (`NOT IN`), and Common Table Expressions (CTEs) with conditional logic (`CASE WHEN`).
*   **Business Value:** Demonstrated ability to audit tied-up warehouse capital, track logistics delivery compliance, and evaluate international market purchasing power.
*   
### Deutsche Zusammenfassung
*   **Projektziel:** Analyse der globalen B2B-Northwind-Datenbank zur Bereitstellung datengestützter Erkenntnisse über Betriebsskalierung, Lieferketteneffizienz, Beschaffungskosten und Speditions-SLAs.
*   **Angewandte Hard Skills:** Multi-Tabellen-Joins (`INNER`, `LEFT`), fortgeschrittene Aggregationen (`COUNT`, `SUM`, `AVG`), Gruppenfilterung (`HAVING`), wiederverwendbare Datenbank-Views (`CREATE VIEW`), Subqueries (`NOT IN`) und Common Table Expressions (CTEs) mit bedingter Logik (`CASE WHEN`).
*   **Geschäftlicher Nutzen:** Nachgewiesene Kompetenz bei der Prüfung von gebundenem Lagerkapital, der Überwachung von Logistik-Liefertreue und der Bewertung internationaler Kaufkraft.

---
## 1. Project Overview & Scenario
This project is an analysis of the Northwind Foods database. Northwind Foods is a global company that imports and exports gourmet delicacies, purchasing products from various suppliers and selling them to corporate B2B clients. The deliveries are transported via freight shipping companies. 

Following a training scenario where the company data is analyzed to understand the operational baseline, my objective is to perform exploratory data analysis using SQL to answer key business questions and identify fundamental metrics.

## 2. Technical Stack & SQL Concepts
*   **Database Engine:** PostgreSQL / MySQL (Python Environment Integration via `database_connection`)
*   **Core SQL Competencies:** Relational Data Auditing, Data Aggregation (`COUNT`).

## 3. Database Schema Overview
This analysis utilizes data from the following core structural tables:
*   `Products`: Contains information about the items sold, tracking IDs, names, suppliers, categories, unit prices, and stock levels.
*   `Customers`: Contains information about corporate B2B clients, their locations, and corporate contact details.
*   `Orders`: Tracks transactional information, including customer references, employee assignments, processing timestamps, and shipping metrics.

---

## 4. Data Exploration & Baseline Metrics

### Aufgabe 1: Total Product Portfolio
**Business Question:** Wie viele Produkte werden von Northwind Foods angeboten? (How many products are offered by Northwind Foods?)

```sql
SELECT 
    COUNT(ProductID) AS number_products
FROM Products;
```
*   **Execution Output:** 77 products
*   **Business Takeaway:** Northwind maintains a concentrated portfolio of 77 unique products, allowing for streamlined product management.

### Aufgabe 2: Corporate Customer Base
**Business Question:** Wie viele Firmenkunden hat Northwind Foods? (How many corporate clients does Northwind Foods have?)

```sql
SELECT 
    COUNT(CustomerID) AS num_customers 
FROM Customers;
```
*   **Execution Output:** 93 customers
*   **Business Takeaway:** The company serves a distinct network of 93 corporate clients, highlighting the importance of individual B2B client management.
### Aufgabe 3: Monthly Transactional Volume (March 1997)
**Business Question:** Wie viele Bestellungen wurden insgesamt im März 1997 aufgegeben? (How many total orders were placed in March 1997?)

```sql
SELECT 
    COUNT(*) AS num_orders_march
FROM Orders
WHERE YEAR(OrderDate) = 1997 
  AND MONTH(OrderDate) = 3;
```
*   **Execution Output:** 30 orders
*   **Business Takeaway:** Isolating specific monthly volumes allows the company to establish seasonal baseline trends and track month-over-month transactional activity.
### Aufgabe 4: Geographic Order Distribution (Multi-Table Joins)
**Business Question:** Wie viele Bestellungen werden in jedes Land geliefert? (How many orders are shipped to each destination country, ordered alphabetically?)

```sql
SELECT 
    COUNT(o.OrderID) AS num_orders,
    c.Country
FROM Orders AS o 
JOIN Customers AS c ON o.CustomerID = c.CustomerID 
GROUP BY c.Country
ORDER BY c.Country ASC;
```

*   **Execution Output (Top 5 Distribution Markets):**
    *   Argentina: 16 orders
    *   Austria: 40 orders
    *   **Belgium: 19 orders (Quiz Milestone Line 3)**
    *   Brazil: 83 orders
    *   Canada: 30 orders
*   **Business Takeaway:** Linking transactional orders to customer geographies highlights regional market volume. This logic establishes which international corridors generate the heaviest shipping traffic, directly affecting logistics planning and freight partner allocations.

---

## 5. Advanced Relational Logic & Data Discrepancies

### The `UnitPrice` Structural Ambiguity
During the schema analysis, a critical structural design choice was identified regarding the `UnitPrice` attribute, which exists in both the `Products` and `Order_Details` tables. Understanding this distinction is vital for accurate financial audits:

*   **`Products.UnitPrice`**: Represents the bulk B2B *purchasing price* (wholesale cost) from suppliers based on the packaging setup defined in `QuantityPerUnit` (e.g., the cost of a full wholesale crate containing 10 cases).
*   **`Order_Details.UnitPrice`**: Represents the final B2B *selling price* (revenue item price) charged directly to the customer for individual units extracted from those packages. 

Maintaining this separation ensures historical sales transactions preserve the exact pricing locked at the time of purchase, independent of any future wholesale supplier price fluctuations.
---

## 6. Financial Revenue Analysis

### Aufgabe 5: Gross Revenue Calculation (Q2-Q3 1997)
**Business Question:** Wie hoch ist der Gesamtumsatz aller Bestellungen im Zeitraum zwischen April und Juli 1997? (What is the total gross revenue generated from all orders between April and July 1997?)

```sql
SELECT 
    SUM(od.Quantity * od.UnitPrice) AS TotalRevenue
FROM Orders AS o
JOIN Order_Details AS od ON o.OrderID = od.OrderID
WHERE o.OrderDate BETWEEN '1997-04-01' AND '1997-07-31';
```

*   **Execution Output:** 207,076.02
*   **Business Takeaway:** By calculating the mathematical product of the selling price and item volume across all order lines within the four-month window, the business can accurately assess gross incoming cash flow. This metric provides a concrete baseline for seasonal performance reviews and financial integration forecasting.
---

## 7. Advanced Analytical Phase: Core Engineering Principles

Transitioning from baseline data exploration to advanced business analysis, all subsequent SQL queries are constructed adhering strictly to enterprise-level development standards and coding best practices:

1.  **Single-Query Architecture:** Resolving complex business requests within a single execution block to maintain pipeline efficiency.
2.  **Divide & Conquer via CTEs:** Utilizing Common Table Expressions (CTEs) instead of deeply nested queries to break complex multi-step data logic into isolated, highly readable intermediate tables.
3.  **Strict Scope Isolation:** Applying clear, descriptive table aliases to secure data lineage across multi-table environments.
4.  **SQL Formatting Standards:** Implementing consistent line breaks, strict indentation, and capitalization patterns (`SELECT`, `JOIN`, `WHERE`) to ensure readability for engineering audits.
5.  **Iterative Workflow Testing:** Developing queries incrementally by reviewing continuous intermediate outputs to choose the optimal logical join types (`INNER` vs. `LEFT JOIN`).

### Schema Expansion: Product Categories
To execute deeper catalog audits, the following relational expansion is implemented:
*   `Categories`: Contains structural tracking columns (`CategoryID`, `CategoryName`, `Description`) and connects directly to the `Products` table via a Foreign Key relationship.
---

## 8. Catalog & Inventory Optimization Analysis

### Aufgabe 6: Product Category & Inventory Representation
**Business Question:** Wie häufig sind die jeweiligen Produktkategorien vertreten? (What is the total product count and sum of warehouse units in stock for each product category, sorted by available inventory descending?)

```sql
SELECT 
    c.CategoryName,
    COUNT(p.ProductID) AS num_Products, 
    SUM(p.UnitsInStock) AS num_units
FROM Products AS p
JOIN Categories AS c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
ORDER BY num_units DESC 
LIMIT 3;
```

*   **Execution Output (Top 3 Heavy Inventory Categories):**
    *   Seafood: 12 products | 701.0 units in stock
    *   Beverages: 12 products | 559.0 units in stock
    *   **Condiments: 12 products | 507.0 units in stock (Quiz Milestone Line 3)**
*   **Business Takeaway:** This multi-aggregate analysis evaluates inventory distribution across product segments. While Seafood, Beverages, and Condiments share an identical count of unique stock-keeping units (12 products each), Seafood carries a significantly higher capital commitment in the warehouse. Tracking this disparity allows supply chain teams to optimize working capital allocations.
### Aufgabe 7: Procurement Asset Valuation & Capital Binding
**Business Question:** Welche Beschaffungskosten stehen hinter den Produktkategorien? (What is the monetary wholesale valuation of the current warehouse inventory, active incoming orders, and the combined overall asset exposure calculated per product category?)

```sql
SELECT 
    c.CategoryName,
    SUM(p.UnitsInStock * p.UnitPrice) AS cost_warehouse,
    SUM(p.UnitsOnOrder * p.UnitPrice) AS cost_orders,
    SUM((p.UnitsInStock + p.UnitsOnOrder) * p.UnitPrice) AS cost_overall
FROM Products AS p
JOIN Categories AS c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
ORDER BY cost_overall DESC 
LIMIT 3;
```
*   **Execution Output (Top 3 Highest Capital Commitments):**
    *   Seafood: Warehouse Cost: 13,010.35 | Order Cost: 1,965.00 | Overall Cost: 14,975.35
    *   Condiments: Warehouse Cost: 12,023.55 | Order Cost: 2,400.00 | Overall Cost: 14,423.55
    *   **Dairy Products: Warehouse Cost: 11,271.20 | Order Cost: 2,785.00 | Overall Cost: 14,056.20 **

---

## 9. Supply Chain & Supplier Network Analysis

### Schema Expansion: Supply Chain Integration
To audit procurement pipelines, we analyze how raw products flow from international vendors into the central warehouse:
*   `Suppliers`: Tracks vendor company profiles, geographical origins, and logistics contact details. It maps directly to the `Products` table via a Foreign Key relationship on `SupplierID`.

### Aufgabe 8: Designing a Reusable Analytics View (Data Architecture)
**Business Question:** Build a consolidated, reusable database View that integrates `Suppliers`, `Products`, and `Categories` to streamline subsequent supply chain and product performance queries.

```sql
CREATE VIEW view_supplier_product_catalog AS
SELECT 
    s.SupplierID,
    s.CompanyName AS SupplierName,
    s.Country AS SupplierCountry,
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

*   **Business Takeaway:** Creating a database View abstracts the underlying table joins into a single virtual table. This strategy improves reporting performance, ensures data consistency across the analytical team, and creates a clean data source that can be easily plugged directly into business intelligence tools.

---

## 9. Supply Chain & Supplier Network Analysis

### Schema Expansion: Supply Chain Integration
To audit procurement pipelines, we analyze how raw products flow from international vendors into the central warehouse:
*   `Suppliers`: Tracks vendor company profiles, geographical origins, and logistics contact details. It maps directly to the `Products` table via a Foreign Key relationship on `SupplierID`.

### Aufgabe 8: Designing a Reusable Analytics View (Data Architecture)
**Business Question:** Erstelle einen View, der die Tabellen Suppliers, Products und Categories miteinander verknüpft. (Build a consolidated, reusable database View that integrates Suppliers, Products, and Categories to streamline subsequent supply chain and product performance queries.)

```sql
CREATE OR REPLACE VIEW suppliersproducts AS (
    SELECT 
        p.ProductName, 
        c.CategoryName,
        p.SupplierID, 
        s.CompanyName, 
        p.UnitPrice
    FROM Suppliers AS s
    JOIN Products AS p ON p.SupplierID = s.SupplierID 
    JOIN Categories AS c ON p.CategoryID = c.CategoryID
);

-- Previewing the View data layer
SELECT * FROM suppliersproducts LIMIT 10;
```

*   **Execution Output (First 10 Rows):**
    *   Chai | Beverages | 1 | Exotic Liquids | 18.0
    *   Chang | Beverages | 1 | Exotic Liquids | 19.0
    *   Guaraná Fantástica | Beverages | 10 | Refrescos Americanos LTDA | 4.5
    *   Sasquatch Ale | Beverages | 16 | Bigfoot Breweries | 14.0
    *   Steeleye Stout | Beverages | 16 | Bigfoot Breweries | 18.0
    *   Côte de Blaye | Beverages | 18 | Aux joyeux ecclésiastiques | 263.5
    *   Chartreuse verte | Beverages | 18 | Aux joyeux ecclésiastiques | 18.0
    *   Ipoh Coffee | Beverages | 20 | Leka Trading | 46.0
    *   Laughing Lumberjack Lager | Beverages | 16 | Bigfoot Breweries | 14.0
    *   Outback Lager | Beverages | 7 | Pavlova, Ltd. | 15.0

*   **Business Takeaway:** Creating a database View abstracts the underlying table joins into a single virtual table. This strategy improves reporting performance, ensures data consistency across the analytical team, and creates a clean data source that can be easily plugged directly into business intelligence tools like Power BI.
### Aufgabe 9: Filtered Portfolio Extraction via View Layer
**Business Question:** Erzeuge eine Liste aller Produktnamen der Kategorie 'Confections' sowie die dazugehörigen Namen der Lieferunternehmen. Sortiere die Liste alphabetisch nach den Produktnamen. (Generate a filtered list of all product names under the 'Confections' category along with their supplier names, ordered alphabetically by product name.)

```sql
SELECT 
    ProductName, 
    CompanyName
FROM suppliersproducts
WHERE CategoryName = 'Confections'
ORDER BY ProductName ASC 
LIMIT 3;
```

*   **Execution Output (Top 3 Alphabetical Confections Vendors):**
    *   Chocolade | Zaanse Snoepfabriek
    *   Gumbär Gummibärchen | Heli Süßwaren GmbH & Co. KG
    *   **Maxilaku | Karkki Oy **
*   **Business Takeaway:** This task demonstrates the operational utility of a view layer. By querying the virtual table directly, we isolate specific category lines and track supplier distribution networks instantly without repeating expensive join instructions.

*   ### Aufgabe 10: Exclusive Vendor Segment Isolation (Subqueries & Set Logic)
**Business Question:** Gib eine Liste aller Lieferunternehmen aus, die Produkte aus der Kategorie 'Grains/Cereals' liefern, aber keine Produkte der Kategorie 'Dairy Products'. Sortiere die Liste alphabetisch nach den Namen der Lieferunternehmen. (Extract a distinct list of all suppliers that deliver products in the 'Grains/Cereals' category, but strictly do not supply any products from the 'Dairy Products' category, ordered alphabetically.)

```sql
SELECT 
    CompanyName, 
    CategoryName,
    ProductName 
FROM suppliersproducts
WHERE CategoryName = 'Grains/Cereals'
  AND CompanyName NOT IN (
      SELECT CompanyName
      FROM suppliersproducts 
      WHERE CategoryName = 'Dairy Products'
  )
ORDER BY CompanyName ASC 
LIMIT 3;
```

*   **Execution Output (Target Vendor Segment):**
    *   G'day, Mate | Grains/Cereals | Filo Mix
    *   Leka Trading | Grains/Cereals | Singaporean Hokkien Fried Mee
    *   **Pasta Buttini s.r.l. | Grains/Cereals | Gnocchi di nonna Alice (Quiz Milestone Line 3)**
*   **Business Takeaway:** This query applies cross-category exclusion logic to isolate specific vendor characteristics. In procurement, identifying suppliers that specialize heavily in one niche (Grains/Cereals) while having zero footprint in another volatile market (Dairy Products) allows risk management teams to map supplier dependencies and negotiate targeted service level agreements (SLAs).
### Aufgabe 11: Multi-Category Vendor Pricing Analysis (The HAVING Clause)
**Business Question:** Gib eine Liste aller Lieferunternehmen aus, die Produkte aus den Kategorien 'Beverages' oder 'Condiments' liefern und deren durchschnittliche Produktpreise mindestens 17 Euro betragen. Sortiere absteigend nach dem Durchschnittspreis. (Extract all suppliers delivering 'Beverages' or 'Condiments' whose average product price is at least 17 Euros, sorted by average price descending.)

```sql
SELECT 
    CompanyName,
    AVG(UnitPrice) AS average_price
FROM suppliersproducts
WHERE CategoryName IN ('Beverages', 'Condiments')
GROUP BY CompanyName
HAVING AVG(UnitPrice) >= 17 
ORDER BY average_price DESC
LIMIT 3;
```

*   **Execution Output (Top 3 Premium Beverage & Condiment Suppliers):**
    *   Aux joyeux ecclésiastiques | 140.750
    *   Leka Trading | 32.725
    *   **Grandma Kelly's Homestead | 32.500 (Quiz Milestone Line 3)**
*   **Business Takeaway:** This query demonstrates advanced aggregated filtering using the `HAVING` clause to isolate premium, high-margin vendors within targeted inventory categories. Identifying suppliers with high average order costs helps finance and procurement teams target the right vendor partnerships for strategic volume discount negotiations.
---

## 10. Logistics Performance & Shipping SLA Audits

### Schema Expansion: Freight Fulfillment
To measure logistical reliability and transportation efficiency, we map the downstream distribution network:
*   `Shippers`: Contains the list of external shipping partners and links directly to the `Orders` ledger where the carrier ID is recorded under the column `ShipVia`.

### Aufgabe 12: Shipping Carrier Fulfillment & Capacity Performance
**Business Question:** Wie ist die Performance der Speditionen? (What is the performance profile of each shipping carrier regarding average fulfillment speed in days, delivery volume, and freight weight distributions?)

```sql
SELECT 
    s.CompanyName AS shipper,
    ROUND(AVG(DATEDIFF(o.ShippedDate, o.OrderDate)), 1) AS avg_diff_order_shipping,
    ROUND(AVG(o.Freight), 1) AS avg_freight,
    ROUND(MIN(o.Freight), 1) AS min_freight,
    ROUND(MAX(o.Freight), 1) AS max_freight,
    COUNT(o.OrderID) AS num_orders
FROM Orders AS o
JOIN Shippers AS s ON o.ShipVia = s.ShipperID
GROUP BY s.CompanyName
ORDER BY s.CompanyName ASC
LIMIT 3;
```

*   **Execution Output (Complete Logistics Portfolio):**
    *   Federal Shipping | Avg Speed: 7.5 days | Avg Freight: 80.4 | Min: 0.4 | Max: 1007.6 | Orders: 255
    *   Speedy Express | Avg Speed: 8.6 days | Avg Freight: 65.0 | Min: 0.1 | Max: 458.8 | Orders: 249
    *   **United Package | Avg Speed: 9.2 days | Avg Freight: 86.6 | Min: 1.4 | Max: 890.8 | Orders: 326 (Quiz Milestone Line 3)**
*   **Business Takeaway:** This service level agreement (SLA) report evaluates logistical bottlenecks. While Federal Shipping provides the fastest turnaround times (7.5 days on average), United Package absorbs the highest logistical stress, moving the largest total package volume (326 orders) and maintaining the heaviest average cargo load capacity (86.6).

---

## 11. Strategic Market Insights & Advanced Analytics Case

### Aufgabe 13: Delivery Delays & Geographic Purchasing Power (M&A Capstone Matrix)
**Business Question:** Welche Lieferverzögerungen gibt es pro Land? (Generate a comprehensive macro-economic overview analyzing market size, total revenue, shipping non-fulfillment rates, and localized shipping compliance percentages for every target country.)

```sql
WITH order_level AS (
    SELECT
        o.OrderID, 
        c.Country, 
        c.CustomerID,
        CASE
            WHEN o.ShippedDate IS NULL THEN 'no_ship'
            WHEN o.ShippedDate < o.RequiredDate THEN 'before'
            WHEN o.ShippedDate = o.RequiredDate THEN 'on'
            ELSE 'after'	
        END AS ship_status,	
        SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS order_revenue 
    FROM Orders o
    JOIN Customers c ON o.CustomerID = c.CustomerID
    LEFT JOIN Order_Details od ON o.OrderID = od.OrderID
    GROUP BY o.OrderID, c.Country, c.CustomerID, ship_status
)
SELECT
    Country,
    COUNT(DISTINCT CustomerID) AS num_cust, 
    COUNT(DISTINCT OrderID) AS num_ord,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'before' THEN 1 END) / COUNT(OrderID), 1) AS perc_ords_before_req,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'after' THEN 1 END) / COUNT(OrderID), 1) AS perc_ords_after_req,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'on' THEN 1 END) / COUNT(OrderID), 1) AS perc_ords_on_req,
    ROUND(100.0 * COUNT(CASE WHEN ship_status = 'no_ship' THEN 1 END) / COUNT(OrderID), 1) AS perc_ords_no_ship,
    ROUND(SUM(order_revenue), 1) AS total_revenue
FROM order_level
GROUP BY Country
ORDER BY total_revenue DESC 
LIMIT 3;
```

*   **Execution Output (Top 3 Highest-Value Global Markets):**
    *   USA | Customers: 13 | Orders: 122 | Before Target: 91.8% | Delayed: 5.7% | On Target: NaN | Lost/Unshipped: 2.5% | Total Revenue: $263,536.9
    *   Germany | Customers: 11 | Orders: 122 | Before Target: 94.3% | Delayed: 3.3% | On Target: 0.8% | Lost/Unshipped: 1.6% | Total Revenue: $244,083.0
    *   **Austria | Customers: 2 | Orders: 40 | Before Target: 92.5% | Delayed: 2.5% | On Target: NaN | Lost/Unshipped: 5.0% | Total Revenue: $139,496.6 (Quiz Milestone Line 3)**
*   **Business Takeaway:** This macro-matrix serves as a strategic corporate guide. By calculating purchase power combined with logistics friction, it reveals that Austria represents an exceptionally high-value market. Despite having only 2 corporate customers, Austria generates more than half the revenue of the entire German market (11 customers). However, Austria's unfulfilled delivery rate (`perc_ords_no_ship`) sits at a high 5.0%, showing an immediate supply chain vulnerability that requires operational correction.
---

## 12. License / Lizenz
This project utilizes the industry-standard Northwind database schema, which is open-source and distributed under the **MIT License**. 

Dieses Projekt verwendet das standardisierte Northwind-Datenbankschema, welches als Open-Source-Software unter der **MIT-Lizenz** bereitgestellt wird.

