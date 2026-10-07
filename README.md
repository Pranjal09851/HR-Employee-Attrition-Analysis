# HR-Employee-Attrition-Analysis

An end-to-end HR analytics project that finds why employees leave and which groups are most at risk. Data is stored and queried in PostgreSQL, loaded into Excel through an ODBC connection (Power Query), and presented as an interactive dashboard built with PivotTables, charts and slicers.

<img width="1406" height="791" alt="image" src="https://github.com/user-attachments/assets/d31d6a06-24a2-4694-b674-de139c31201d" />


📌 Business Questions
What is the overall attrition rate, and how many employees left?
Which departments, job roles and age groups lose the most people?
Does overtime, pay, marital status or business travel affect attrition?
Do people leave early in their tenure?
Where should HR focus retention efforts?
🗂️ Dataset
Detail	Value
Source	IBM HR Analytics Employee Attrition & Performance (Kaggle, fictional dataset)
Employees	1,470
Original columns	35 (age, department, job role, monthly income, overtime, marital status, tenure, satisfaction scores, etc.)
Target column	attrition (Yes / No)
🏗️ Workflow
CSV  ──►  PostgreSQL (hr database, table: hr_attrition)
                │
                ├──►  SQL queries: KPIs, segment analysis, ranking
                │
                └──►  Excel (ODBC + Power Query, refreshable)
                          │
                          ├──  Helper columns
                          ├──  9 PivotTables
                          └──  Dashboard: 8 charts · 4 KPI cards · 3 slicers
🛠️ Tools & Skills
PostgreSQL: aggregation, FILTER, CASE WHEN, window functions (RANK() OVER), GROUP BY
ODBC / Power Query: live connection from Excel to PostgreSQL
Excel: PivotTables, PivotCharts, slicers, linked KPI cards, custom dashboard background
Excel formulas: IFS, helper columns
📊 Dashboard
KPI	Value
Total Employees	1,470
Employees Left	237
Attrition Rate	16.12%
Avg Monthly Income	6,503

Charts: Attrition rate by Job Role · Age Group · Department · Overtime · Income Band · Marital Status · Tenure Band, plus Share of Leavers by Gender. Slicers: Department · Gender · Business Travel. They filter every chart and KPI together.

Technique used: a helper column attrition_flag (1 if the employee left, else 0). In a PivotTable, the average of this column is the attrition rate for any group, and the sum is the number of leavers.

Other helper columns: age_group, income_band, tenure_band (built with IFS).

🔍 Key Insights
Overall attrition is 16.12% (237 of 1,470 employees).
Overtime is the strongest driver. Employees working overtime leave at 30.5% vs 10.4% for those who don't. They are 28% of staff but 54% of all leavers.
Early-tenure risk: employees with 0–1 years at the company leave at 34.9% (75 of 215), falling to about 10–11% after 5 years.
Young employees leave most: under-25s have a 39.2% attrition rate; ages 35–54 are about 10%.
Pay matters: attrition is 28.6% for incomes below 3,000 vs 8.9% for 10,000+. Leavers earned 4,787 on average vs 6,833 for those who stayed.
Role hotspots: Sales Representatives lose 39.8% (33 of 83), followed by Laboratory Technicians (23.9%) and HR (23.1%). Research Directors lose only 2.5%.
Department: Sales 20.6%, Human Resources 19.0%, Research & Development 13.8%.
Single employees leave at 25.5% vs 10.1% for divorced and 12.5% for married. Single employees who also work overtime leave at 49.6%.
Frequent business travellers leave at 24.9% vs 8.0% for non-travellers.
Gender is not a strong factor: male 17.0% vs female 14.8%. Men are 63% of leavers largely because they are 60% of staff.
💡 Recommendations
Review overtime workload and compensation, especially in Sales and Laboratory roles.
Strengthen onboarding and mentoring in the first two years.
Revisit pay for the lowest income band and for Sales Representatives.
Build career paths for under-25 employees.
Reduce travel load for frequent travellers or compensate for it.
🧮 SQL (PostgreSQL)

Table: hr_attrition. Full script with table setup: sql/hr_attrition_queries.sql

sql
-- Overall attrition rate (%)
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE attrition = 'Yes') / COUNT(*), 2) AS attrition_rate_pct
FROM hr_attrition;

-- Attrition by job role, ranked by risk
SELECT job_role,
       COUNT(*)                                  AS employees,
       COUNT(*) FILTER (WHERE attrition = 'Yes') AS employees_left,
       ROUND(100.0 * COUNT(*) FILTER (WHERE attrition = 'Yes') / COUNT(*), 2) AS attrition_rate_pct,
       RANK() OVER (ORDER BY 1.0 * COUNT(*) FILTER (WHERE attrition = 'Yes') / COUNT(*) DESC) AS risk_rank
FROM hr_attrition
GROUP BY job_role
ORDER BY risk_rank;

-- Attrition by age group
SELECT CASE WHEN age < 25 THEN 'Under 25'
            WHEN age BETWEEN 25 AND 34 THEN '25-34'
            WHEN age BETWEEN 35 AND 44 THEN '35-44'
            WHEN age BETWEEN 45 AND 54 THEN '45-54'
            ELSE '55+' END AS age_group,
       COUNT(*) AS employees,
       ROUND(100.0 * COUNT(*) FILTER (WHERE attrition = 'Yes') / COUNT(*), 2) AS attrition_rate_pct
FROM hr_attrition
GROUP BY 1
ORDER BY MIN(age);

-- Impact of overtime
SELECT over_time,
       COUNT(*) AS employees,
       ROUND(100.0 * COUNT(*) FILTER (WHERE attrition = 'Yes') / COUNT(*), 2) AS attrition_rate_pct
FROM hr_attrition
GROUP BY over_time;

-- Leavers vs stayers
SELECT attrition,
       ROUND(AVG(monthly_income), 0)     AS avg_monthly_income,
       ROUND(AVG(years_at_company), 1)   AS avg_tenure_years,
       ROUND(AVG(job_satisfaction), 2)   AS avg_job_satisfaction
FROM hr_attrition
GROUP BY attrition;
📁 Repository Structure
hr-attrition-analysis/
├── README.md
├── sql/
│   └── hr_attrition_queries.sql
├── excel/
│   └── HR_Attrition_Dashboard.xlsx
├── data/
│   └── hr_attrition.csv
└── images/
    └── dashboard.png
▶️ How to Reproduce
Create the hr database and the hr_attrition table in PostgreSQL, then import the CSV (pgAdmin → Import/Export).
Run the queries in sql/ to explore the data.
Install the psqlODBC driver. In Excel go to Data → Get Data → From Other Sources → From ODBC, connect to localhost:5432, database hr, and load public.hr_attrition.
Open the Excel dashboard and use Data → Refresh All to update the PivotTables, charts and KPIs.
