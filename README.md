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
```python
order_table['order_date'] = pd.to_datetime(order_table['order_date'])
order_table['quarter']    = order_table['order_date'].dt.quarter

order_item = pd.merge(order_table, item_table, on='order_id', how='left')
order_item.groupby('quarter')['line_total'].sum()
```
### Output:
| Quarter | Total Revenue |
| ----- | ----- |
| Q1  | $1,915.93 |
| Q2 | $1,258.44 |
| Q3 | $506.00 |
| Q4 | $394.50 |

### Findings: Revenue dropped 79% from Q1 to Q4, confirming a significant and sustained decline across the year.

### Step 2: Volume vs Basket Size
To determine whether the drop was caused by fewer orders or smaller orders, both metrics were calculated separately:
```python
# Completed order count per quarter
order_item[order_item['order_status'] == 'Completed']\
    .groupby('quarter')['order_id'].nunique()

# Average basket size per quarter
revenue     = order_item.groupby('quarter')['line_total'].sum()
total_order = order_item.groupby('quarter')['order_id'].nunique()
basket_size = revenue / total_order
```
### Output:
| Quarter | Completed orders | Avg Basket Size |
| ------ | ------ | -------|
| Q1 | 16 | $137.53 |
| Q2 | 12 | $117.45 |
| Q3 | 12 | $68.42 |
| Q4 | 12 | $39.38 | 
### Findings: Order volume dropped once in Q2 then stabilised. Basket size, however, fell every single quarter, losing 71% of its value by Q4. The revenue drop is a basket size problem, not a volume problem.

### Step 3: Category Breakdown
The products table was merged in to enable category-level analysis:
```pyhton
df = pd.merge(product_table, order_item, on='product_id', how='right')

completed_order = df[df['order_status'] == 'Completed']
completed_order.groupby(['quarter', 'category'])['line_total'].sum()
```
### Output
| Quarter | Accessories | Electronics | Furniture |
| ------ | ------ | ------ | ------ |
| Q1 | $350.50 | $1,329.93 | $236.00 |
| Q2 | $188.50 | $959.94 | $110.00 |
| Q3 | $275.00 | $65.00 | $166.00 |
| Q4 | $118.50 |  |  |
### Finding:  Electronics accounted for the vast majority of revenue in Q1 and Q2. It collapsed in Q3 and vanished entirely in Q4. By Q4, only low-value Accessories orders remained explaining the basket size crash.

### Step 4: Why Did Electronics Stop Selling?
#### 4a: Order Status Breakdown for Electronics
```python
df[(df['category'] == 'Electronics')].groupby(['quarter', 'order_status'])['order_id'].nunique()
```
#### Output:
| Quarter | Completed | Returned |
| --- | ----- | ------ |
| Q1 | 10 | 1 |
| Q2 | 6 | 1 |
| Q3 | 1 | 2 |
| Q4 | 0 | 0 | 
#### Finding: By Q3, more Electronics orders were being returned than completed; a 67% return rate. By Q4, no Electronics orders were placed at all.

#### 4b: Discount Coverage for Electronics
```python
df2 = pd.merge(df, discount_table, on='product_id', how='left')

df2[df2['category'] == 'Electronics']\
    .groupby('quarter')['discount_pct'].mean()
```
#### Output 
| Quarter | Avg Discount |
| ---- | ------ |
| Q1 | 9.95% |
| Q2 | 10.58% |
| Q3 | 13.75% |
| Q4 | |
#### Finding: Despite increasing discount rates through Q3, sales kept falling. No discount was offered on Electronics in Q4 at all. Hence, discounts were not the cause of the recovery failure.

#### 4C:  Product-Level Breakdown
```python
df[df['category'] == 'Electronics']\
    .groupby(['quarter', 'product_name'])['line_total'].sum()
```
#### Output
| Product | Q1 | Q2 | Q3 | Q4 |
|----- | ----- | ----- | ----- | ------ |
| Wireless Earbuds Pro | $449.95 | $359.96 | | |
| Noise-Cancel Headphones | $399.98 | $199.99 | | | 
| Mechanical Keyboard | $240.00 | $120.00 | | | 
| Webcam HD 1080p | $195.00 | $130.00 | $65.00 | |
| Gaming Headset RGB | $149.99 | $149.99 | $149.99 | |
| Portable SSD 1TB | $95.00 | $95.00 | $95.00 | |
#### Finding: Finding: The top three Electronics products (Earbuds, Headphones, Keyboard) collectively generated $1,089.93 in Q1 but completely disappeared after Q2. The remaining products faded through Q3 before the entire category went silent in Q4.

## Key Findings 
| # | Finding | Impact |
| --- | ---- | ---- |
| 1. | Revenue fell 79% from Q1 to Q4 | Critical |
| 2. | Order volume stabilised after Q2; basket size drove the decline | High |
| 3. | Electronics category disappeared entirely by Q4 | Critical |
| 4. | Top 3 Electronics products vanished after Q2 | Critical |
| 5. | Return rate for Electronics reached 67% in Q3 | High |
| 6. | Discounts on Electronics increased but failed to reverse the trend | Medium |
| 7. | No Electronics discount was offered in Q4 | Medium |

## Conclusion
The Q4 revenue drop was not caused by fewer customers; order volume held steady from Q2 onward. The root cause was the complete collapse of the Electronics category, which was responsible for the majority of revenue in Q1 and Q2.

The three highest-value Electronics products (Wireless Earbuds Pro, Noise-Cancel Headphones, Mechanical Keyboard) stopped generating sales after Q2, and by Q3 more Electronics orders were being returned than completed. With Electronics gone and only low-value Accessories remaining, average basket size crashed from $137 in Q1 to $39 in Q4; a 71% decline.









