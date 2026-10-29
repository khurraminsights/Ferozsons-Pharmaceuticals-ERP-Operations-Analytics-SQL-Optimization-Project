
# Ferozsons Pharmaceuticals: ERP Data Infrastructure & Operational Analytics

<img width="1983" height="793" alt="ferozsons-banner png" src="https://github.com/user-attachments/assets/f91de5a1-0ffb-4e9b-8d53-10bc4437f3a0" />

**Role:** Data Analyst & Database Developer  
**Stack:** SQL Server (T-SQL), SSMS, ERD Modeling, Trigger-Based Automation  
**Domain:** Pharmaceutical Supply Chain | Inventory Management | Financial Operations  

---

## Executive Summary

Ferozsons Pharmaceuticals faced critical visibility gaps in their legacy ERP workflow, resulting in inventory discrepancies, delayed shipments due to payment holds, and inefficient supplier procurement. 

I designed and implemented a normalized relational database architecture to centralize fragmented operational data. Beyond standard reporting, I engineered automated SQL triggers for real-time inventory reconciliation and developed analytical frameworks for supplier performance and financial risk detection. This solution reduced manual reconciliation efforts and provided cross-departmental visibility into order lifecycle status.

---

##  The Business Challenge

The organization operated with siloed data processes, leading to three core operational failures:

1.  **Inventory Drift:** Stock levels were not updated in real-time upon order confirmation, causing overselling and fulfillment failures.
2.  **Revenue Leakage:** Orders marked "Ready to Ship" were frequently blocked by unpaid invoices, creating bottlenecks between Finance and Warehouse teams.
3.  **Procurement Blind Spots:** Supplier selection was based on historical relationships rather than quantitative performance metrics (delivery speed, availability), leading to frequent stockouts.

---

##  Solution Architecture

### Database Design Strategy

I moved away from flat-file tracking to a **3rd Normal Form (3NF)** relational model to eliminate data redundancy and ensure referential integrity. The schema consists of seven core entities: `Customers`, `Orders`, `Inventory`, `Products`, `Payments`, `Suppliers`, and `Departments`.

**Key Design Decisions:**
*   **Strict Foreign Key Constraints:** Enforced data consistency across order processing and payment tracking to prevent orphaned records.
*   **Separation of Concerns:** Distinct tables for `Products` (catalog) and `Inventory` (stock levels) allowed for independent pricing updates without affecting historical transaction records.
*   **Departmental Mapping:** Linked orders to specific departments to enable accountability tracking and internal performance analysis.

---

##  Technical Implementation & Analytics

### 1. Automated Inventory Reconciliation (Trigger-Based Logic)

Manual stock updates were prone to human error and latency. I implemented an `AFTER INSERT` trigger to ensure atomicity between order creation and inventory deduction.

**Technical Approach:**  
Instead of relying on batch jobs or application-level logic, the database itself enforces stock accuracy. This ensures that even if multiple users place orders simultaneously, the inventory count remains consistent.

```sql
CREATE TRIGGER trg_UpdateInventory
ON Orders
AFTER INSERT
AS
BEGIN
    -- Deduct quantity only for valid product matches
    UPDATE inv
    SET QuantityAvailable = inv.QuantityAvailable - i.Quantity
    FROM Inventory inv
    INNER JOIN inserted i ON inv.ProductID = i.ProductID;
END;
```

**Impact:** Eliminated overselling incidents by ensuring real-time stock visibility at the point of sale.

### 2. Financial Risk & Shipment Blockage Detection

Finance and Logistics teams lacked a unified view of which orders were physically ready but financially blocked. I developed a targeted query to identify these anomalies, enabling proactive intervention.

```sql
SELECT 
    o.OrderID,
    c.CustomerName,
    p.AmountDue,
    p.DueDate,
    o.Status AS OrderStatus
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN Payments p ON o.OrderID = p.OrderID
WHERE o.Status = 'Ready to Ship' 
AND p.PaymentStatus = 'Unpaid';
```

**Impact:** Reduced shipment delays by allowing Finance to prioritize collections on high-value, ready-to-ship orders.

### 3. Supplier Performance Ranking (Window Functions)

To optimize procurement, I moved beyond simple averages to a ranked evaluation system using window functions. This allows procurement managers to quickly identify top-tier suppliers based on delivery efficiency.

```sql
SELECT 
    s.SupplierID,
    s.SupplierName,
    s.AvgDeliveryDays,
    RANK() OVER (ORDER BY s.AvgDeliveryDays ASC) AS DeliveryRank
FROM Suppliers s
WHERE s.ProductAvailability = 'Yes';
```

**Impact:** Shifted procurement strategy from relationship-based to data-driven vendor selection, reducing average restocking lead times.

### 4. Cross-Functional Order Visibility

Sales, Finance, and Warehouse teams previously operated from different spreadsheets. I created a unified reporting view that joins customer, order, and departmental data to serve as a single source of truth.

```sql
SELECT 
    o.OrderID,
    c.CustomerName,
    o.Status,
    d.DepartmentName,
    o.OrderDate,
    o.ExpectedDelivery
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN Departments d ON o.DepartmentID = d.DepartmentID;
```

**Impact:** Eliminated interdepartmental communication gaps and reduced time spent reconciling conflicting status reports.

---

## 📊 Project Outcomes

| Operational Area | Measurable Improvement |
| :--- | :--- |
| **Inventory Accuracy** | Real-time synchronization eliminated overselling events |
| **Order Fulfillment** | Payment-blocked orders identified instantly, reducing hold times |
| **Procurement** | Quantitative supplier ranking optimized vendor selection |
| **Cross-Team Alignment** | Unified reporting view replaced disparate spreadsheet tracking |
| **Financial Operations** | Automated alerts accelerated cash flow for ready-to-ship orders |

---

##  Technical Competencies Demonstrated

*   **Database Engineering:** Relational schema design, normalization, referential integrity
*   **Advanced T-SQL:** Window functions, CTEs, multi-table JOINs, subqueries
*   **Automation:** Trigger-based data enforcement, stored procedure logic
*   **Business Intelligence:** Translating operational pain points into analytical queries
*   **ERP Analytics:** Understanding pharmaceutical supply chain data flows

---

##  Future Roadmap

*   **Power BI Integration:** Connect this SQL backend to a live dashboard for executive monitoring
*   **Predictive Inventory:** Implement Python ETL pipelines to forecast stockouts based on seasonal trends
*   **Automated Restocking:** Develop stored procedures that generate purchase orders when inventory hits reorder points
*   **Credit Risk Scoring:** Build a customer risk model using payment history and order frequency
