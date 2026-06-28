# Data Cleaning Log — raw_orders.xlsx

**Source file:** raw_orders.xlsx  
**Output file:** cleaned_orders.xlsx  
**Date:** 2026-06-28  

---

## 1. Whitespace Cleaning (TRIM / SUBSTITUTE)
- Applied to every cell in every column.
- Removed leading, trailing, and internal extra spaces.
- Examples caught: `"  North "`, `" Small Business "`, `"Paid "`, `"Corporate "`, `"  Furniture "`.

---

## 2. Case Standardisation (Find & Replace)
- `segment`, `region`, `category`, `sub_category`, `ship_mode` → Title Case.
- Fixed values: `STANDARD CLASS` → `Standard Class`, `FIRST CLASS` → `First Class`, `NORTH` → `North`, `storage` → `Storage`.

---

## 3. Date Standardisation (Flash Fill logic)
- Six mixed date formats detected and unified to `YYYY-MM-DD`:
  - `21 Jul 2024`, `07/27/2024`, `28-11-2024`, `2024-05-24`, `07 Feb 2024`, `29 Apr 2024`
- Stored in `order_date_clean` and `ship_date_clean` columns.

---

## 4. Duplicate Handling
- **Exact duplicates (all columns identical):** 20 rows removed (kept first occurrence).
- **Conflicting duplicates (same order_id, different values):** 12 order IDs flagged as `CONFLICTING_DUPLICATE` — retained both rows for manual review.

---

## 5. Missing Value Handling
| Column | Action |
|---|---|
| `region` | Filled with `"Unknown"` and flagged `MISSING_REGION` |
| `ship_mode` | Filled with `"Unknown"` and flagged `MISSING_SHIP_MODE` |
| `discount` | Filled with `0` only where all other sales fields are valid; flagged `MISSING_DISCOUNT` otherwise |

---

## 6. Calculated Columns Added
| Column | Formula / Logic |
|---|---|
| `cleaned_discount` | Standardised discount (0 for missing, flagged if negative or > 0.5) |
| `calculated_sales` | `quantity × unit_price × (1 − discount)` |
| `calculated_profit` | `calculated_sales − cost` |
| `profit_margin` | `calculated_profit / calculated_sales` |
| `shipping_delay_days` | `ship_date − order_date` in calendar days |
| `order_month` | Month + year extracted from `order_date` (e.g. `Jul 2024`) |
| `order_year` | Year extracted from `order_date` (e.g. `2024`) |
| `data_quality_flag` | Pipe-delimited string of applicable flags (see below) |

---

## 7. Data Quality Flags
| Flag | Count | Description |
|---|---|---|
| `MISSING_REGION` | 25 | Region was blank — filled with "Unknown" |
| `MISSING_SHIP_MODE` | 21 | Ship mode was blank — filled with "Unknown" |
| `MISSING_DISCOUNT` | 26 | Discount blank with otherwise valid sales data |
| `NEGATIVE_DISCOUNT` | 15 | Discount value is negative (invalid) |
| `HIGH_DISCOUNT` | 7 | Discount exceeds 50% (suspicious) |
| `SHIP_BEFORE_ORDER` | 21 | Ship date is earlier than order date (data entry error) |
| `CONFLICTING_DUPLICATE` | 24 | Same order_id with different field values across rows |

---

## 8. Assumptions & Decisions
- Negative discounts were **not corrected** — flagged only; manual review required.
- Conflicting duplicate rows were **both retained** and flagged rather than arbitrarily dropping one.
- `calculated_sales` and `calculated_profit` are preferred over the raw `sales` and `profit` columns for analysis, as the raw values show discrepancies in some flagged rows.
- Records with `order_status` of Cancelled, Returned, or payment_status of Failed/Refunded should be **excluded from completed-sales summaries** per business rules.
