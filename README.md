# Modelling-Project
# Retail Sales & Marketing Analytics — Data Model

A star-schema data model built to unify **sales, order fulfillment, inventory, marketing campaigns, and revenue targets** into a single analytics-ready structure. Designed for BI reporting and self-service dashboarding (Power BI).

![Data Model Diagram](./data_model_diagram.png)

## Project Overview

This model consolidates data from multiple operational sources — order management, CRM/customer master, inventory, and campaign platforms — into a single semantic layer. The goal was to answer business questions that normally require joining several disconnected systems, such as:

- What's our revenue and discount trend by channel, region, and product category?
- How is order fulfillment performing (order → invoice → ship → delivery → pay cycle)?
- Which campaigns are driving product-level sales, and what's the cost per outcome?
- Are we tracking against sales targets by period?
- How does inventory availability correlate with sales and promotions?

## Why This Design: Star Schema with Multiple Fact Tables

Rather than a single flat fact table, I used **multiple fact tables sharing conformed dimensions** (`dim_products`, `dim_date`, `dim_CUSTMASTER`, `dim_city`, `dim_CAMPAIGN`). This is a deliberate design choice:

- Each fact table represents a distinct **business process** at its own natural grain (one row per order line, per promotion-product mapping, per campaign-day, etc.) — this avoids grain-mixing errors and keeps aggregations accurate.
- Shared dimensions let all facts be sliced consistently by the same product, customer, date, and geography attributes — enabling cross-process analysis (e.g., "campaign spend vs. actual product sales") without duplicating dimension data.
- A `security` table (Region + UserEmail) is included to support **row-level security**, restricting report access by region per user — a common enterprise requirement.

## Model Structure

### Fact Tables (business processes / transactions)

| Fact Table | Grain | Purpose |
|---|---|---|
| `fact_order` | One row per order line | Core sales transactions — pricing, discount %, quantity, line total |
| `fact_order_process` | One row per order | Order lifecycle timestamps (order → invoice → ship → deliver → pay) for fulfillment/SLA analysis |
| `fact_inventory` | One row per product | Stock/unit availability by product |
| `fact_promotion_coverage` | One row per campaign-product mapping | Which products were covered under which promotional campaign |
| `fact_CAMPAIGN_spend` | One row per campaign-day | Daily marketing spend, clicks, and impressions per campaign |
| `fact_sales_targets` | One row per period | Target revenue by period, for actual-vs-target tracking |

### Dimension Tables (descriptive context)

| Dimension | Key Attributes |
|---|---|
| `dim_products` | Brand, Category, Subcategory, Primary Supplier, Product Code |
| `dim_CUSTMASTER` | Customer name, Account Manager, Region, Segment, Payment Terms, Contact info |
| `dim_city` | City, Region, geo key |
| `dim_date` | Date, Month, Year — standard date table for time intelligence |
| `dim_CAMPAIGN` | Campaign name, Channel, Budget, Start/End Date |
| `Dim_orders` | Order channel code/name, priority, source system |
| `security` | Region-based access mapping for row-level security |

## Relationships

All fact tables connect to shared dimensions via surrogate keys (`Product_key`, `geo_key`, `CustomerID`, `campaign_id`, `Date`), forming a classic **snowflake-leaning star schema**. `dim_date` and `dim_products` act as the primary conformed dimensions, linking sales, inventory, and campaign activity for cross-functional analysis.

## Tools & Skills Demonstrated

- **Dimensional modeling**: star schema design, grain definition, conformed dimensions, surrogate keys
- **Power BI**: data model relationships, row-level security setup
- **Business process mapping**: translating order-to-cash and campaign-to-sale workflows into a queryable structure
- **Data governance**: source file tracking (`SourceFile` field), region-based access control

## Next Steps / Possible Extensions

- Add a `fact_returns` table for returns/refunds analysis
- Extend `dim_date` with fiscal calendar attributes
- Build out DAX measures layer (YoY growth, campaign ROI, fulfillment SLA %)

---
*Feel free to explore the `.pbix` file / model screenshots in this repo for the full implementation.*
