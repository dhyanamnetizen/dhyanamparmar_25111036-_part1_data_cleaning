# Raw Orders Data Cleaning & Analysis Project

---

## 1. Problem Summary

The raw orders dataset (`raw_orders.xlsx`) contained significant data quality issues that made it unreliable for business analysis. Date values were stored in at least **six different formats**, text fields had inconsistent casing and hidden whitespace, discount values included negatives and blanks, and the dataset contained both exact duplicate rows and conflicting duplicate order IDs.

The goal of this project was to:
- Clean and standardise the raw data using Excel techniques (TRIM, SUBSTITUTE, Flash Fill, Text to Columns, Find & Replace)
- Flag and document all data quality issues without silently dropping records
- Add calculated columns to support downstream analysis
- Produce pivot summary reports covering sales performance, profitability, shipping, and problem orders

---

## 2. Dataset Description

| Attribute | Detail |
|---|---|
| **Source file** | `raw_orders.xlsx` |
| **Sheet** | `raw_orders` |
| **Original row count** | 932 (including duplicates) |
| **Cleaned row count** | 912 |
| **Columns (raw)** | 21 |
| **Columns (cleaned)** | 28 (21 original + 7 calculated) |
| **Date range** | January 2024 – October 2025 |
| **Geographies** | 5 regions, 15+ Indian states |
| **Customer segments** | Consumer, Corporate, Home Office, Small Business |
| **Product categories** | Furniture, Office Supplies, Technology |
| **Order statuses** | Completed, Returned, Cancelled |
| **Payment statuses** | Paid, Pending, Refunded, Failed |

---

## 3. Tools Used

| Tool / Technique | Purpose |
|---|---|
| **Python (pandas)** | Data loading, transformation, aggregation |
| **openpyxl** | Excel file creation, formatting, styling |
| **TRIM / SUBSTITUTE logic** | Strip whitespace and fix spacing in all cells |
| **Find & Replace (case fix)** | Normalise mixed-case values (`NORTH` → `North`, `STANDARD CLASS` → `Standard Class`) |
| **Flash Fill (date parsing)** | Detect and standardise 6 mixed date formats → `YYYY-MM-DD` |
| **Text to Columns logic** | Parse quality flags into a pipe-delimited `Quality Flag` column |

---

## 4. Cleaning Steps Performed

### Step 1 — Whitespace (TRIM / SUBSTITUTE)
Applied to every cell. Removed leading, trailing, and internal extra spaces.  
*Examples caught:* `"  North "`, `" Small Business "`, `"Paid "`, `"Corporate "`, `"  Furniture "`

### Step 2 — Case Standardisation (Find & Replace)
Title-cased `Segment`, `Region`, `Category`, `Sub-Category`, and `Ship Mode`.  
*Fixed values:* `STANDARD CLASS` → `Standard Class`, `FIRST CLASS` → `First Class`, `NORTH` → `North`, `storage` → `Storage`

### Step 3 — Date Standardisation (Flash Fill logic)
Detected and unified **6 mixed date formats** into `YYYY-MM-DD`:
- `21 Jul 2024` → `2024-07-21`
- `07/27/2024` → `2024-07-27`
- `28-11-2024` → `2024-11-28`
- `2024-05-24` → unchanged
- `07 Feb 2024` → `2024-02-07`
- `29 Apr 2024` → `2024-04-29`

### Step 4 — Duplicate Handling
- **20 exact duplicate rows** removed (all columns identical — kept first occurrence)
- **12 order IDs** with conflicting field values were retained and flagged `CONFLICTING_DUPLICATE` for manual review

### Step 5 — Missing Value Imputation
| Column | Action |
|---|---|
| `Region` | Filled blank cells with `"Unknown"` and flagged `MISSING_REGION` |
| `Ship Mode` | Filled blank cells with `"Unknown"` and flagged `MISSING_SHIP_MODE` |
| `Discount` | Filled with `0` where all other sales fields were valid; flagged `MISSING_DISCOUNT` otherwise |

### Step 6 — Calculated Columns Added
| Column | Formula |
|---|---|
| `Calc. Sales (₹)` | `Quantity × Unit Price × (1 − Discount)` |
| `Calc. Profit (₹)` | `Calc. Sales − Cost` |
| `Profit Margin` | `Calc. Profit / Calc. Sales` |
| `Ship Delay (Days)` | `Ship Date − Order Date` in calendar days |
| `Order Month` | Month + Year extracted from Order Date (e.g. `Jul 2024`) |
| `Order Year` | Year extracted from Order Date |
| `Quality Flag` | Pipe-delimited string of all applicable flags per row |

---

## 5. Business Rules Applied

| Rule | Action Taken |
|---|---|
| Do not overwrite raw file | Created `cleaned_orders.xlsx` as a separate output |
| Missing region | Filled as `"Unknown"` and flagged |
| Missing ship mode | Filled as `"Unknown"` and flagged |
| Missing discount | Treated as `0` only if all other sales fields are valid; otherwise flagged |
| Negative discounts | Flagged `NEGATIVE_DISCOUNT` — NOT auto-corrected |
| High discounts (>50%) | Flagged `HIGH_DISCOUNT` for review |
| Cancelled / Failed / Refunded | Excluded from all completed-sales pivot summaries (Pivots 1, 2, 4, 6) |
| Date sequence | Flagged `SHIP_BEFORE_ORDER` where ship date < order date |
| Exact duplicates | Removed; IDs logged in quality report |
| Conflicting duplicates | Retained both rows; flagged `CONFLICTING_DUPLICATE` |

---

## 6. Summary of Data Quality Issues Found

| Flag | Count | Description |
|---|---|---|
| `MISSING_REGION` | 25 | Region was blank |
| `MISSING_SHIP_MODE` | 21 | Ship mode was blank |
| `MISSING_DISCOUNT` | 26 | Discount blank with otherwise valid sales data |
| `NEGATIVE_DISCOUNT` | 15 | Discount value is negative (data entry error) |
| `HIGH_DISCOUNT` | 7 | Discount exceeds 50% (suspicious) |
| `SHIP_BEFORE_ORDER` | 21 | Ship date is earlier than order date |
| `CONFLICTING_DUPLICATE` | 24 | Same order ID with different field values across rows |
| **Exact duplicate rows removed** | **20** | All columns identical — removed before flagging |

**Total flagged rows (post-dedup):** ~119 unique rows carry at least one quality flag.  
Flagged rows are highlighted in **amber** in `cleaned_orders.xlsx` and detailed in the `quality_report` sheet.

---

## 7. Summary of Final Pivot Reports (`pivot_summary.xlsx`)

| Sheet | Description | Sort / Filter |
|---|---|---|
| `1_Sales_Profit_Region` | Total sales & profit by region | ⚑ Sorted by Total Sales ↓ |
| `2_Category_SubCategory` | Sales & profit by category and sub-category | ⚑ Sorted: Category A→Z, then Sales ↓ |
| `3_Order_Count_ShipMode` | Order count & avg shipping delay by ship mode | Sorted by Order Count ↓ |
| `4_Margin_by_Segment` | Avg profit margin by customer segment | ⚑ Sorted by Avg Margin ↓ |
| `5_Problem_Orders_Region` | Refunded / cancelled / failed orders by region | ⚑ Filtered to problem orders only |
| `6_Monthly_Sales_Trend` | Monthly sales & profit trend (completed orders) | Chronological order |

All pivot sheets include a title row, source filter note, data rows, and a grand total row.

---

## 8. Key Business Insights

1. **South is the top-performing region** with ₹16,04,234 in completed sales, followed by West and North.

2. **Technology drives revenue** — it is the highest-grossing category at ₹22,24,748 in completed sales, ahead of Furniture and Office Supplies.

3. **Overall profit margin is 29.1%** across completed orders, indicating healthy but improvable margins.

4. **Home Office segment has the best profit margin** at 30.4%, slightly outperforming other segments.

5. **Problem orders are significant** — 310 orders (34% of all orders) were returned, cancelled, refunded, or failed, representing ₹31,09,109 in lost sales value. This warrants investigation.

6. **21 orders shipped before their order date** — a data entry or system sync issue that should be investigated with the source system.

7. **15 orders have negative discounts** — these inflate reported sales artificially and need correction at the source.

8. **Conflicting duplicate order IDs (12 IDs, 24 rows)** suggest a data pipeline issue where the same order ID was recorded with different values (e.g., different sales figures or statuses).

---

## 9. Assumptions and Limitations

- **Negative discounts** were flagged but not auto-corrected, as the correct value is unknown. They are excluded from analysis recommendation.
- **Conflicting duplicates** were both retained — the "correct" record cannot be determined programmatically. Manual review or source system lookup is required.
- **Missing discount** was assumed to be `0` only where `Calc. Sales`, `Cost`, and `Quantity` were all present and valid; otherwise flagged.
- **`Calc. Sales` and `Calc. Profit`** are used as the authoritative values for all pivot analysis, as the raw `sales` and `profit` columns contained inconsistencies in flagged rows.
- **"Problem orders"** are defined as: `Order Status` ∈ {Returned, Cancelled} **OR** `Payment Status` ∈ {Refunded, Failed}. Some orders may satisfy both conditions and are counted once.
- Date parsing assumes Indian locale conventions where ambiguous (e.g., `01/07/2024` is treated as MM/DD/YYYY based on majority format pattern in the dataset).
- The dataset covers orders from 2024–2025 only; no historical benchmarking is possible.

---

## 10. Screenshots

> Screenshots of key outputs should be placed in the `screenshots/` folder and referenced below.

| File | Must Show |
|---|---|
| `screenshots/cleaned_orders_sample.png` | First 20 rows of `cleaned_orders.xlsx` with Quality Flag column visible |
| `screenshots/quality_report.png` | The `quality_report` sheet showing flag summary table |
| `screenshots/pivot1_region.png` | Pivot 1 — Sales & Profit by Region (sorted) |
| `screenshots/pivot5_problem.png` | Pivot 5 — Problem orders by Region (filtered, amber rows) |
| `screenshots/pivot6_trend.png` | Pivot 6 — Monthly Sales Trend table |

---

## Output Files

| File | Description |
|---|---|
| `cleaned_orders.xlsx` | Main cleaned dataset with 28 columns, quality flags, and calculated fields |
| `pivot_summary.xlsx` | 6-sheet pivot summary workbook |
| `cleaning_log.md` | Detailed cleaning decisions and assumptions log |
| `README.md` | This document |
