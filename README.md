# 🏭 SQL Project — Evaluate a Manufacturing Process
## 📌 Executive Summary

This project evaluates the stability and quality of a manufacturing process using SQL-based Statistical Process Control (SPC). By analyzing historical measurements from multiple machines (operators), the project calculates rolling averages, standard deviations, and dynamic control limits — Upper Control Limit (UCL) and Lower Control Limit (LCL) — for each part produced.

Using SQL window functions, the system monitors the last five measurements of each operator to detect deviations in real time. Any part whose height falls outside the calculated control limits is automatically flagged as an out-of-control event.

This approach provides a data-driven method to ensure consistent product quality, identify equipment issues early, and improve operational efficiency. The result is a scalable quality-monitoring SQL solution that can be applied across manufacturing lines to support continuous improvement and reduce defect rates.

---

## 📊 Objectives

### 1️⃣ Compute Control Limits (UCL & LCL)
Using a rolling window of 5 items per machine (`operator`), calculate:
- `avg_height`
- `stdev_height`
- `ucl = avg_height + 3 × (stdev_height / √n)`
- `lcl = avg_height − 3 × (stdev_height / √n)`

### 2️⃣ Identify Out-of-Control Parts
Flag any item where:
- `height > ucl`  
- `height < lcl`

### 3️⃣ Produce a Final Table With Alerts
Include for each item:
- operator  
- item number  
- height  
- rolling average + stdev  
- UCL & LCL  
- alert (TRUE/FALSE)

---

## 🗂️ Dataset

The dataset comes from the `manufacturing_parts` table with the following fields:

| Column     | Description                    |
|------------|--------------------------------|
| item_no    | Serial number of the part      |
| length     | Length measurement             |
| width      | Width measurement              |
| height     | Height measurement             |
| operator   | Machine that produced the item |

---

## 🧠 SQL Concepts Used

- `ROW_NUMBER()` for tracking sequence per machine  
- Rolling window calculations using `ROWS BETWEEN 4 PRECEDING AND CURRENT ROW`  
- Statistical functions: `AVG()`, `STDDEV()`  
- Control limit calculations (UCL & LCL)  
- Conditional CASE logic for anomaly detection  
- CTE structure for readability

---

## 🧾 Final SQL Query

```sql
-- Step 1: Create row numbers per operator
WITH ranked AS (
    SELECT
        operator,
        item_no,
        height,
        ROW_NUMBER() OVER (
            PARTITION BY operator 
            ORDER BY item_no
        ) AS row_number
    FROM manufacturing_parts
),

-- Step 2: Rolling statistics over last 5 rows
stats AS (
    SELECT
        operator,
        item_no,
        height,
        AVG(height) OVER (
            PARTITION BY operator 
            ORDER BY item_no 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) AS avg_height,
        STDDEV(height) OVER (
            PARTITION BY operator 
            ORDER BY item_no 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) AS stdev_height,
        COUNT(*) OVER (
            PARTITION BY operator 
            ORDER BY item_no 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) AS n_window_5
    FROM ranked
)

-- Step 3: Calculate UCL/LCL + produce alert flags
SELECT
    operator,
    item_no,
    height,
    avg_height,
    stdev_height,
    (avg_height + (3 * stdev_height / SQRT(n_window_5))) AS ucl,
    (avg_height - (3 * stdev_height / SQRT(n_window_5))) AS lcl,
    CASE
        WHEN height > (avg_height + (3 * stdev_height / SQRT(n_window_5))) THEN TRUE
        WHEN height < (avg_height - (3 * stdev_height / SQRT(n_window_5))) THEN TRUE
        ELSE FALSE
    END AS alert
FROM stats
WHERE n_window_5 = 5;
```

## 📈 Example Output Columns

| operator | item_no | height | avg_height | stdev_height | ucl   | lcl   | alert |
|----------|---------|--------|------------|---------------|--------|--------|--------|
| Op1      | 15      | 20.25  | 20.00      | 0.40          | 21.20 | 18.80 | TRUE   |
| Op2      | 9       | 18.90  | 19.10      | 0.30          | 20.00 | 18.20 | FALSE  |


(table is illustrative)

## 🚀 Insights & Value
- Automatically detects quality deviations

- Enables early intervention before defects increase

- Ensures parts consistently meet design specifications

- Provides operators with feedback on machine performance

- Demonstrates practical SQL data quality monitoring skills

## 📝 Summary
This project applies SQL window functions to evaluate a manufacturing process using SPC methodology.
Rolling statistics, control limits, and alert detection combine to provide a real-time quality monitoring system.
