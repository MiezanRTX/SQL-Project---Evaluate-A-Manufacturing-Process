\# 🏭 SQL Project — Evaluate a Manufacturing Process



This project analyzes manufacturing part measurements using SQL window functions and Statistical Process Control (SPC).  

The goal is to calculate rolling control limits (UCL \& LCL) for part height measurements and flag any items that fall outside the acceptable range.



---



\## 📊 Objectives



\### 1️⃣ Compute Control Limits (UCL \& LCL)

Using a rolling window of 5 items per machine (`operator`), calculate:

\- `avg\_height`

\- `stdev\_height`

\- `ucl = avg\_height + 3 × (stdev\_height / √n)`

\- `lcl = avg\_height − 3 × (stdev\_height / √n)`



\### 2️⃣ Identify Out-of-Control Parts

Flag any item where:

\- `height > ucl`  

\- `height < lcl`



\### 3️⃣ Produce a Final Table With Alerts

Include for each item:

\- operator  

\- item number  

\- height  

\- rolling average + stdev  

\- UCL \& LCL  

\- alert (TRUE/FALSE)



---



\## 🗂️ Dataset



The dataset comes from the `manufacturing\_parts` table with the following fields:



| Column     | Description                    |

|------------|--------------------------------|

| item\_no    | Serial number of the part      |

| length     | Length measurement             |

| width      | Width measurement              |

| height     | Height measurement             |

| operator   | Machine that produced the item |



---



\## 🧠 SQL Concepts Used



\- `ROW\_NUMBER()` for tracking sequence per machine  

\- Rolling window calculations using `ROWS BETWEEN 4 PRECEDING AND CURRENT ROW`  

\- Statistical functions: `AVG()`, `STDDEV()`  

\- Control limit calculations (UCL \& LCL)  

\- Conditional CASE logic for anomaly detection  

\- CTE structure for readability



---



\## 🧾 Final SQL Query



```sql

-- Step 1: Create row numbers per operator

WITH ranked AS (

&nbsp;   SELECT

&nbsp;       operator,

&nbsp;       item\_no,

&nbsp;       height,

&nbsp;       ROW\_NUMBER() OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no

&nbsp;       ) AS row\_number

&nbsp;   FROM manufacturing\_parts

),



-- Step 2: Rolling statistics over last 5 rows

stats AS (

&nbsp;   SELECT

&nbsp;       operator,

&nbsp;       item\_no,

&nbsp;       height,

&nbsp;       AVG(height) OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no 

&nbsp;           ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

&nbsp;       ) AS avg\_height,

&nbsp;       STDDEV(height) OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no 

&nbsp;           ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

&nbsp;       ) AS stdev\_height,

&nbsp;       COUNT(\*) OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no 

&nbsp;           ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

&nbsp;       ) AS n\_window\_5

&nbsp;   FROM ranked

)



-- Step 3: Calculate UCL/LCL + produce alert flags

SELECT

&nbsp;   operator,

&nbsp;   item\_no,

&nbsp;   height,

&nbsp;   avg\_height,

&nbsp;   stdev\_height,

&nbsp;   (avg\_height + (3 \* stdev\_height / SQRT(n\_window\_5))) AS ucl,

&nbsp;   (avg\_height - (3 \* stdev\_height / SQRT(n\_window\_5))) AS lcl,

&nbsp;   CASE

&nbsp;       WHEN height > (avg\_height + (3 \* stdev\_height / SQRT(n\_window\_5))) THEN TRUE

&nbsp;       WHEN height < (avg\_height - (3 \* stdev\_height / SQRT(n\_window\_5))) THEN TRUE

&nbsp;       ELSE FALSE

&nbsp;   END AS alert

FROM stats

WHERE n\_window\_5 = 5;

```



📈 Example Output Columns

operator	item\_no	height	avg\_height	stdev\_height	ucl	lcl	alert

Op1	15	20.25	20.00	0.40	21.20	18.80	TRUE

Op2	9	18.90	19.10	0.30	20.00	18.20	FALSE



(table is illustrative)



🚀 Insights \& Value

Automatically detects quality deviations



Enables early intervention before defects increase



Ensures parts consistently meet design specifications



Provides operators with feedback on machine performance



Demonstrates practical SQL data quality monitoring skills



📝 Summary

This project applies SQL window functions to evaluate a manufacturing process using SPC methodology.

Rolling statistics, control limits, and alert detection combine to provide a real-time quality monitoring system.

