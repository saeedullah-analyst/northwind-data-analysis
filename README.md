# northwind-data-analysis
SQL data analysis project for Northwind Foods post-merger business integration.
# SQL Project: Northwind Foods Data Analysis Case Study

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
