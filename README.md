
# Ferozsons Pharmaceuticals: ERP Data Infrastructure & Operational Analytics

<img width="1983" height="793" alt="ferozsons-banner png" src="https://github.com/user-attachments/assets/f91de5a1-0ffb-4e9b-8d53-10bc4437f3a0" />

# Pharmaceutical ERP Operations & SQL Analytics

### Database Design | Inventory Management | Financial Controls | Procurement Intelligence

![SQL Server](https://img.shields.io/badge/SQL_Server-T--SQL-CC2927?style=for-the-badge)
![Database](https://img.shields.io/badge/Database-3NF_Design-0078D4?style=for-the-badge)
![Analytics](https://img.shields.io/badge/Analytics-Operational_Intelligence-00A6A6?style=for-the-badge)
![Status](https://img.shields.io/badge/Project-Portfolio_Case_Study-5865F2?style=for-the-badge)

**An SQL Server project exploring how relational database design, inventory automation, and analytical reporting can improve pharmaceutical ERP workflows.**

[Project Overview](#project-overview) •
[Architecture](#solution-architecture) •
[SQL Implementation](#technical-implementation) •
[Business Value](#business-value) •
[Roadmap](#future-roadmap)

</div>

---

## Project Overview

| Attribute | Details |
|---|---|
| **Project** | Pharmaceutical ERP Operations Analytics |
| **Role** | Data Analyst & Database Developer |
| **Primary Technology** | Microsoft SQL Server |
| **Query Language** | T-SQL |
| **Development Environment** | SQL Server Management Studio (SSMS) |
| **Database Design** | Relational Modeling, 3NF, Primary & Foreign Keys |
| **SQL Techniques** | JOINs, Window Functions, Triggers |
| **Business Domain** | Pharmaceutical Supply Chain & ERP Operations |
| **Project Type** | Independent Portfolio Case Study |

> **Project disclaimer:** This is an independent portfolio case study using a pharmaceutical ERP business scenario. It should not be interpreted as a verified description of Ferozsons Pharmaceuticals' internal systems, operational problems, financial performance, or business outcomes. Example metrics and illustrations are not actual company results.

---

## Executive Summary

Pharmaceutical distributors depend on accurate inventory records, reliable order processing, and coordination between procurement, finance, and warehouse operations.

Disconnected business systems can create operational challenges such as inaccurate stock balances, payment-related shipment holds, and limited visibility into supplier performance.

This project demonstrates an SQL Server-based approach to addressing these challenges through relational database architecture and operational analytics.

### What I Developed

- A normalized relational database structure supporting core ERP entities.
- SQL queries for identifying unpaid, shipment-ready orders.
- Trigger-based inventory update logic as a foundation for automated stock management.
- Supplier performance rankings using SQL window functions.
- Multi-table reporting queries for cross-departmental order visibility.

### Analytical Objective

Transform operational ERP records into structured information that can support more reliable reporting, inventory monitoring, financial controls, and procurement decisions.

---

## The Business Problem

The project addresses four operational challenges commonly encountered in ERP environments.

| Business Challenge | Operational Risk | Proposed Analytical Solution |
|---|---|---|
| Inventory inconsistency | Incorrect stock balances and potential overselling | Automated inventory updates and validation |
| Payment-related shipment holds | Orders remain blocked despite being ready for dispatch | SQL-based financial exception reporting |
| Limited supplier visibility | Procurement decisions lack consistent performance comparisons | Supplier delivery rankings |
| Fragmented reporting | Departments rely on inconsistent order information | Centralized relational reporting |

### Key Business Questions

1. How can inventory quantities be updated when new orders are created?
2. Which shipment-ready orders have outstanding payments?
3. Which available suppliers have the shortest average delivery times?
4. How can order, customer, and departmental records be combined for reporting?
5. What additional database controls are required to maintain reliable ERP information?

---

## Solution Architecture

### Relational Database Design

The proposed database follows Third Normal Form (3NF) principles to reduce unnecessary duplication and support referential integrity.

### Core Database Entities

| Entity | Purpose |
|---|---|
| `Customers` | Customer information and identifiers |
| `Orders` | Sales transactions and order status |
| `Products` | Product master information |
| `Inventory` | Product stock availability |
| `Payments` | Payment information and outstanding balances |
| `Suppliers` | Supplier information and delivery attributes |
| `Departments` | Organizational ownership and reporting |

### Conceptual Data Flow

```text
              PHARMACEUTICAL ERP SYSTEM
                         |
          +--------------+--------------+
          |                             |
     MASTER DATA                  TRANSACTION DATA
          |                             |
    +-----+------+              +-------+-------+
    |            |              |               |
 Customers    Products        Orders          Payments
    |            |              |
    |        Inventory          |
    |            |              |
    +------------+--------------+
                 |
          SQL SERVER DATABASE
                 |
       +---------+----------+
       |         |          |
    Inventory  Financial  Supplier
    Analysis   Controls   Analysis
       |         |          |
       +---------+----------+
                 |
          BUSINESS REPORTING
```

*Conceptual architecture illustrating the intended analytical workflow.*

### Database Design Principles

**1. Data Normalization**

Separate master records from operational transactions to improve consistency and reduce redundant attributes.

**2. Referential Integrity**

Use primary and foreign key constraints to maintain valid relationships between business entities.

**3. Separation of Responsibilities**

Maintain independent product catalog and inventory structures so that stock updates do not unnecessarily alter product master information.

**4. Cross-Functional Reporting**

Connect orders, customers, payments, suppliers, and departments to support reporting across business functions.

---

# Technical Implementation

## 01 — Inventory Automation Using SQL Triggers

### Business Requirement

Inventory must reflect order quantities reliably, without depending entirely on manual updates.

### SQL Implementation

The following example demonstrates a set-based trigger that deducts quantities from matching inventory records when orders are inserted.

```sql
CREATE TRIGGER trg_UpdateInventory
ON Orders
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    UPDATE inv
    SET inv.QuantityAvailable =
        inv.QuantityAvailable - src.TotalQuantity
    FROM Inventory AS inv
    INNER JOIN
    (
        SELECT
            ProductID,
            SUM(Quantity) AS TotalQuantity
        FROM inserted
        GROUP BY ProductID
    ) AS src
        ON inv.ProductID = src.ProductID;
END;
```

### Technical Explanation

The `inserted` pseudo-table stores rows associated with the current insert operation.

Rather than assuming only one order is inserted, the trigger aggregates quantities by product.

The inventory update then matches those product IDs against the inventory table.

### Important Production Considerations

This implementation demonstrates inventory deduction, but it is **not a complete overselling-prevention mechanism**.

A production solution should additionally validate:

- Sufficient available stock before deduction.
- Missing inventory records.
- Invalid or negative order quantities.
- Order cancellation and return adjustments.
- Warehouse-level inventory allocation.
- Appropriate transaction isolation and concurrency controls.

### Business Value

Provides a foundation for automated inventory synchronization and reduces reliance on manual stock adjustment.

---

## 02 — Financial Risk & Shipment Exception Analysis

### Business Requirement

Finance and warehouse teams need to identify orders that are ready for dispatch but have outstanding payments.

### SQL Implementation

```sql
SELECT
    o.OrderID,
    c.CustomerName,
    p.AmountDue,
    p.DueDate,
    o.Status AS OrderStatus
FROM Orders AS o

INNER JOIN Customers AS c
    ON o.CustomerID = c.CustomerID

INNER JOIN Payments AS p
    ON o.OrderID = p.OrderID

WHERE
    o.Status = 'Ready to Ship'
    AND p.PaymentStatus = 'Unpaid'

ORDER BY
    p.AmountDue DESC,
    p.DueDate ASC;
```

### Analytical Logic

The query combines three business entities:

- `Orders` provides dispatch readiness.
- `Customers` identifies the account.
- `Payments` provides outstanding payment details.

It filters unpaid orders marked ready for shipment and prioritizes larger outstanding balances.

### Business Interpretation

This report can help finance teams prioritize payment follow-ups and identify orders requiring financial clearance.

### Expected Benefit

Improved visibility into financial exceptions affecting fulfillment workflows.

> **Note:** The query depends on the payment status model. Systems allowing multiple partial payments may require an aggregated outstanding-balance calculation.

---

## 03 — Supplier Performance Analysis Using Window Functions

### Business Requirement

Procurement teams need a consistent method for comparing suppliers based on delivery speed and product availability.

### SQL Implementation

```sql
SELECT
    s.SupplierID,
    s.SupplierName,
    s.AvgDeliveryDays,

    RANK() OVER (
        ORDER BY s.AvgDeliveryDays ASC
    ) AS DeliveryRank

FROM Suppliers AS s

WHERE
    s.ProductAvailability = 'Yes'
    AND s.AvgDeliveryDays IS NOT NULL

ORDER BY
    DeliveryRank,
    s.SupplierName;
```

### Technical Explanation

The `RANK()` window function assigns supplier rankings based on average delivery duration.

Suppliers with equal delivery times receive the same rank.

The query excludes unavailable suppliers and records with missing delivery-time values.

### Analytical Interpretation

Lower average delivery times indicate faster historical supplier delivery performance.

However, delivery speed should not be the only procurement criterion.

A comprehensive supplier scorecard should also consider:

- Product availability.
- On-time delivery percentage.
- Cost competitiveness.
- Order fulfillment reliability.
- Product quality and compliance.

### Expected Benefit

Provides a transparent basis for supplier comparisons and procurement planning.

---

## 04 — Cross-Functional Order Reporting

### Business Requirement

Operational departments need consistent access to order information without repeatedly reconciling separate reports.

### SQL Implementation

```sql
SELECT
    o.OrderID,
    c.CustomerName,
    o.Status AS OrderStatus,
    d.DepartmentName,
    o.OrderDate,
    o.ExpectedDelivery

FROM Orders AS o

INNER JOIN Customers AS c
    ON o.CustomerID = c.CustomerID

INNER JOIN Departments AS d
    ON o.DepartmentID = d.DepartmentID

ORDER BY
    o.OrderDate DESC,
    o.OrderID;
```

### Analytical Explanation

This query combines order, customer, and department information into a single reporting dataset.

It supports questions such as:

- Which department is associated with each order?
- What is the current order status?
- Which orders have upcoming expected delivery dates?
- How is order activity distributed across departments?

### Expected Benefit

More consistent reporting across operational teams and reduced manual reconciliation effort.

---

## Business Value

The project demonstrates how SQL can support four important ERP capabilities.

| Area | Implemented Analytical Capability | Potential Business Benefit |
|---|---|---|
| Inventory | Set-based trigger logic | More timely stock updates |
| Finance | Unpaid order identification | Faster exception identification |
| Procurement | Supplier delivery rankings | More informed supplier comparison |
| Operations | Cross-functional JOIN queries | Consistent order visibility |
| Database Management | Relational schema design | Improved structural data consistency |

**Important:** These are expected or potential benefits. Quantified improvements in inventory accuracy, shipment time, costs, or financial performance have not been established through before-and-after measurements in this README.

---

## Technical Skills Demonstrated

<table>
<tr>
<td width="50%" valign="top">

### Database Development

- Relational schema design
- Third Normal Form (3NF)
- Primary and foreign keys
- Entity relationship modeling
- Data integrity considerations

</td>
<td width="50%" valign="top">

### SQL Analytics

- Multi-table JOINs
- SQL window functions
- Filtering and aggregation
- Operational exception reporting
- Business-focused query design

</td>
</tr>
<tr>
<td valign="top">

### Automation

- SQL Server triggers
- Set-based inventory updates
- Transactional data considerations
- Validation requirements

</td>
<td valign="top">

### Business Analysis

- Inventory management
- Procurement performance
- Payment exception analysis
- ERP reporting workflows
- Cross-functional data interpretation

</td>
</tr>
</table>

---

## Future Roadmap

The following capabilities are planned enhancements, not claims of completed implementation.

| Priority | Enhancement | Intended Outcome |
|---|---|---|
| 1 | Power BI Integration | Interactive operational monitoring |
| 2 | Inventory Validation | Stronger stock and transaction controls |
| 3 | Automated Reordering | Replenishment recommendations using reorder thresholds |
| 4 | Supplier Scorecards | Multi-factor supplier evaluation |
| 5 | Python Forecasting | Experimental inventory demand forecasting |
| 6 | Customer Credit Risk | Payment history and outstanding-balance analysis |

### Planned Business Intelligence Dashboard

A future Power BI implementation could include:

**Inventory KPIs**
- Available stock
- Low-stock products
- Inventory movement trends

**Financial KPIs**
- Outstanding payments
- Unpaid shipment-ready orders
- Overdue payment balances

**Procurement KPIs**
- Average supplier delivery time
- Supplier availability
- Procurement performance rankings

**Operations KPIs**
- Orders by status
- Department-level order volumes
- Pending deliveries

---

## Repository Structure

The project may be organized using the following structure as additional artifacts become available.

```text
ferozsons-erp-analytics/
|
|-- README.md
|-- ferozsons-banner.png
|
|-- database/
|   |-- schema.sql
|   |-- constraints.sql
|
|-- sql/
|   |-- inventory_automation.sql
|   |-- financial_exceptions.sql
|   |-- supplier_performance.sql
|   |-- operational_reporting.sql
|
|-- documentation/
|   |-- data_dictionary.md
|   |-- business_requirements.md
|
|-- screenshots/
```

*Suggested organization only; the diagram does not assert that all these files currently exist in the repository.*

---

## Key Takeaways

This project demonstrates the relationship between database engineering and business analysis.

Relational modeling creates a consistent structure for ERP information. SQL triggers can support automated data updates, while analytical queries help identify operational exceptions and compare performance across business entities.

For a production environment, these techniques should be supported by transaction controls, testing, security policies, and measurable operational KPIs.

---

## Connect & Explore

**Khurram Naveed**  
Data Analyst | SQL Server | Business Intelligence

[![GitHub](https://img.shields.io/badge/GitHub-khurraminsights-181717?style=for-the-badge&logo=github)](https://github.com/khurraminsights)

[![Portfolio](https://img.shields.io/badge/View-Analytics_Portfolio-0078D4?style=for-the-badge)](https://github.com/khurraminsights?tab=repositories)

---

<div align="center">

### From Structured Data to Better Operational Decisions

**SQL Server • Database Design • ERP Analytics • Business Intelligence**

Independent data analytics portfolio project.

</div>
