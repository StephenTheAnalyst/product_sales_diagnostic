# Product Sales Diagnostic Analysis
Diagnostic analysis using Python and pandas to investigate a 79% revenue drop in Q4 2023

## Project Overview
This project performs a diagnostic analysis on a retail product sales dataset to answer one core business question: Why did revenue drop in Q4 2023?. Using Python and pandas, the analysis peels back the data layer by layer, from overall revenue trends down to specific products to pinpoint the root cause of the decline. 

## Dataset 
The dataset is a custom-built multi-table Excel workbook simulating onr year of e-commerce sales data across four related tables:
| Table | Rows | Key Columns |
| ----- | ----- | ------  |
| products | 15 | product_id, category, unit_price, is_active |
| orders | 52 | order_id, order_date, region, channel, order_status |
| order_items | 59 | order_id, product_id, quantity, unit_price, line_total |
| discounts | 15 | product_id, campaign_name, discount_pct, start_date, end_date |

* Categories: Electronics, Accessories, Furniture
* Time Period: January 2023 - December 2023
* Regions: North, South, East, West
* Channels: Online, Mobile App, In-Store

## Diagnostic Questions
The analysis follows a structured root-cause investigation across four layers:
1. Layer 1 — Confirm the revenue drop: Is Q4 revenue actually lower?
2. Layer 2 — Volume vs. basket size: Is it fewer orders or smaller orders?
3. Layer 3 — Category breakdown: Which category is driving the decline?
4. Layer 4 — Product level: Why did Electronics stop selling?

## Tools & Libraries 
| Library | Purpose |
| ----- | -------- |
| Pandas | Data loading, merging, filtering, and groupby aggregations |
| Matplotlib | Imported for potential visualisation support |
| Openpyxl | Reading multi-sheet of Excel workbook |

## Analysis Walkthrough

### Step 1: Confirm the Revenue Drop 
After loading all four sheets and merging orders with order_items, quarterly revenue was calculated:












