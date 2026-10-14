# fae-beauty-bi-analytics
Interactive 3-page Power BI dashboard for D2C brand Fae Beauty analyzing shade returns, customer retention, and warehouse inventory deadstock.
# 💄 Fae Beauty: D2C E-Commerce Return & Operations Analytics


---

## 📌 Executive Summary
**Fae Beauty (Free And Equal)** is an inclusive Indian direct-to-consumer (D2C) cosmetic brand creating beauty products tailored specifically for diverse Indian skin tones. 

As a digital-first e-commerce brand operating via web channels and regional fulfillment centers, scaling order volume brought critical operational challenges:
1. **High Return-to-Origin (RTO) and refund drains** driven by customer online shade mismatches.
2. **Customer retention vs. regional non-delivery risks** tied to unverified Cash-on-Delivery (COD) orders.
3. **Frozen working capital in warehouse deadstock** paired with transit lead-time delays.

This project delivers an interactive, 3-page Business Intelligence dashboard developed in **Microsoft Power BI** to bridge product R&D, digital content marketing, retention finance, and supply chain logistics into one actionable interface.

---

## 🏢 Business Problems

| # | Operational Bottleneck | Direct Business Impact | Power BI Analytical Focus |
|---|------------------------|------------------------|---------------------------|
| **1** | **Shade Mismatch Returns** | High RTO logistics overhead and direct refund losses. | Undertone matrix heatmap & video swatch engagement scatter analysis. |
| **2** | **COD Default & Customer Churn** | Unverified cash orders driving failed deliveries; high CAC. | Customer Lifetime Value (CLV) split & regional payment method audit. |
| **3** | **Warehouse Deadstock** | Capital locked in aging stock (>90 days); delivery transit delays. | SKU aging alerts, hub inventory imbalances, and carrier transit metrics. |

---

## 🛠️ Data Architecture & Star Schema Model

### 1. Extraction, Transformation, and Loading (Power Query ETL)
* Handled string casing variations across `shade_family`, `category`, and `undertone` fields.
* Extracted delivery turnaround metrics: `[delivery_days_elapsed] = [delivery_date] - [order_date]`
* Cleaned null values and standardized order/refund transaction statuses (`Approved`, `Pending`, `Rejected`).

### 2. Relational Data Modeling (Star Schema)
The data model connects central transactional fact tables to lookup dimension tables using **1-to-Many single-directional filter flows**:
* **Fact Tables:** `fact_sales_order_product`, `fact_page_views`, `fact_inventory`, `Fact_Returns_&_Feedback`
* **Dimension Tables:** `dim_product`, `dim_customer`, `dim_locations`, `dim_date`, `dim_warehouse`

### 3. Key DAX Calculations
```dax
-- 1. Return Rate %
Return_rate_% = 
DIVIDE (
    CALCULATE ( COUNTROWS ( 'fact_sales_order_product' ), 'fact_sales_order_product'[refund_status] = "Approved" ),
    COUNTROWS ( 'fact_sales_order_product' ),
    0
)

-- 2. Average Swatch Video Engagement Rate
Avg_Swatch_Watch_Rate = 
AVERAGE ( 'fact_page_views'[swatch_%_views] )

-- 3. Overstocked Inventory Flag (>90 Days Lag)
Stock_Status = 
SWITCH (
    TRUE (),
    'fact_inventory'[Current_Stock_Qty] > 300 && DATEDIFF('fact_inventory'[Last_Restock_Date], TODAY(), DAY) > 90, "OVERSTOCKED",
    'fact_inventory'[Current_Stock_Qty] < 50, "LOW STOCK",
    "OPTIMAL"
)
