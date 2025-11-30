🏭 SQL Project: Evaluate a Manufacturing Process



This project analyzes a manufacturing process using SQL window functions, summary statistics, and statistical process control (SPC) principles.

The goal is to calculate the acceptable height range using Upper Control Limit (UCL) and Lower Control Limit (LCL), then flag any manufactured parts that fall outside this acceptable range.



📘 Project Overview



Manufacturing processes require tight control to ensure products meet quality standards.

This project applies SPC methods to determine whether parts produced by different machines (operator) fall within a defined acceptable height range.



According to the project specification (see page 1 of the PDF) :



UCL = avg\_height + 3 × (stdev\_height / √n)



LCL = avg\_height − 3 × (stdev\_height / √n)



Where n is a rolling window of 5 parts.



We use SQL window functions to:



Calculate rolling averages and standard deviations



Compute UCL and LCL dynamically



Identify parts violating control limits



Output a final DataFrame (alerts) containing flagged items



🗂️ Dataset



The data is stored in the manufacturing\_parts table (page 1) :



Field	Description

item\_no	Serial number of the item produced

length	Length of the item

width	Width of the item

height	Height of the item

operator	The operating machine

🧠 SQL Concepts Demonstrated



Window Functions



ROW\_NUMBER()



AVG() OVER()



STDDEV() OVER()



Sliding window frames (ROWS BETWEEN 4 PRECEDING AND CURRENT ROW)



Statistical Process Control (SPC)



Dynamic calculations of control limits



Conditional logic for anomaly detection



CTE structuring for clarity



🧾 Final SQL Solution



This is the full working SQL solution shown on page 2 of the PDF — rewritten cleanly for Markdown:



-- Step 1: Create subquery for row\_number per operator

WITH ranked AS (

&nbsp;   SELECT

&nbsp;       operator,

&nbsp;       item\_no,

&nbsp;       height,

&nbsp;       ROW\_NUMBER() OVER (PARTITION BY operator ORDER BY item\_no) AS row\_number

&nbsp;   FROM public.manufacturing\_parts

),



-- Step 2: Create subquery for rolling statistics (avg, stdev, n=5)

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

&nbsp;       

&nbsp;       STDDEV(height) OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no 

&nbsp;           ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

&nbsp;       ) AS stdev\_height,

&nbsp;       

&nbsp;       COUNT(height) OVER (

&nbsp;           PARTITION BY operator 

&nbsp;           ORDER BY item\_no 

&nbsp;           ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

&nbsp;       ) AS n\_window\_5

&nbsp;   FROM ranked

)



-- Final Output: Calculate UCL/LCL and determine alerts

SELECT

&nbsp;   operator,

&nbsp;   item\_no,

&nbsp;   height,

&nbsp;   avg\_height,

&nbsp;   stdev\_height,

&nbsp;   (avg\_height + (3 \* (stdev\_height) / SQRT(n\_window\_5))) AS ucl,

&nbsp;   (avg\_height - (3 \* (stdev\_height) / SQRT(n\_window\_5))) AS lcl,

&nbsp;   CASE

&nbsp;       WHEN height > (avg\_height + (3 \* (stdev\_height) / SQRT(n\_window\_5))) THEN TRUE

&nbsp;       WHEN height < (avg\_height - (3 \* (stdev\_height) / SQRT(n\_window\_5))) THEN TRUE

&nbsp;       ELSE FALSE

&nbsp;   END AS alert

FROM stats

WHERE n\_window\_5 = 5;



📈 Example Output



Page 2 of the PDF shows the resulting DataFrame :



Multiple measurements per operator



Computed rolling averages \& standard deviations



Dynamically calculated UCL \& LCL



alert = TRUE when height is out of control range



This helps identify operators or batches requiring quality intervention.



🚀 Business Impact



This SQL-based SPC system:



Detects deviations in real time



Ensures consistent product quality



Reduces defects and scrap rates



Helps operators adjust machines before major failures occur



Enables long-term trend analysis for continuous improvement



📌 Summary



This project shows how to combine SQL analytics with SPC methodology to evaluate and monitor a manufacturing process.

By using rolling window calculations, dynamic UCL/LCL, and anomaly detection, we can ensure manufacturing quality stays within acceptable limits.

