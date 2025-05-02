select * from "Cleaned_SaaS_Financial_Model_data.csv";
--pulling in data reviewing for accuracy. 

--Calculating Churn rate (monthly) as a variable churn_rate
SELECT  
    MONTH,
    "Churned Customers",
    "Total Customers",
    Round("Churned Customers" * 100.0 / "Total Customers", 2) as Churn_Rate_Percent
FROM "Cleaned_SaaS_Financial_Model_data"

--Calculating cumulative MRR growth as variable cumulative_mrr
SELECT  
    Month,
    SUM("MRR ($)") OVER (ORDER BY Month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS Cumulative_MRR
FROM "Cleaned_SaaS_Financial_Model_data"

--Creating new table encompassing all data and new columns
SELECT
    base.MONTH,
    base."New Customers",
    base."Churned Customers",
    base."Net Customers",
    base."Total Customers",
    base."ARPU ($)",
    base."MRR ($)",
    base."ARR ($)",
    cum.cumulative_MRR,
    churn.Churn_rate_percent
FROM Cleaned_SaaS_Financial_Model_data AS base
LEFT JOIN Cumulative_MRR AS cum
    ON base.Month = cum.Month
LEFT JOIN Churn_rate AS churn
    ON base.Month = churn.Month
