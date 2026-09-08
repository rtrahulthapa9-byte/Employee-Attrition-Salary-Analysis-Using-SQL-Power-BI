# 📢 Employee Attrition & Salary Analysis


#### Project Overview
This project analyzes HR data covering 1,000 employees across 8 departments and 32 job roles, involving:
* Database Creation — PostgreSQL
* Business Analysis — 10 SQL queries answering key HR questions
* Visualization — Interactive Power BI dashboard

#### Key Analysis Areas 
* Analyze department-wise average salary to identify the highest and lowest paying departments
* Identify employees earning above the company-wide average salary
* Identify employees earning above their own department's average salary
* Determine the highest-paid employee within each department
* Analyze salary hike percentage trends across job roles
* Track year-wise attrition trends to identify improving or worsening retention
* Examine the relationship between performance ratings and attrition
* Identify the tenure stage at which employees are most likely to leave
* Rank job roles by attrition to highlight high-risk roles
* Analyze attrition patterns across age groups

#### Tool I Used:
* PostgreSQL
* Power Bi+DAX
#### SQL Concepts Used
* Aggregate Functions – AVG(), COUNT(), ROUND()
* GROUP BY – summarizing data by Department, Job Role, Year, etc.
* ORDER BY – sorting results ascending/descending
* Subqueries – scalar (company-wide avg) and correlated (department-wise avg)
* CTEs (WITH clause) – precomputing department averages and attrition counts
* JOINs – combining HR_Data with CTE results
* Window Functions – DISTINCT ON, DENSE_RANK() OVER (ORDER BY ...)
* Conditional Aggregation – COUNT(*) FILTER (WHERE ...)
* Calculated Metrics – percentage calculations using 100.0 * for float division
* Aliasing – column aliases (AS) and table aliases (h1, h2)



### 10 Business Questions Answered Using SQL 
##### Q1: What is the average salary in each department, and which departments pay the highest & lowest on average?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/1348eb9b1e8e10a602679ce27379ee38af069a18/Screenshot/Screenshot%202026-09-08%20133617.png)

```bash
SELECT 
     Department, round(AVG(Salary),2) as avg_salary 
FROM HR_Data
GROUP BY Department
ORDER BY avg_salary DESC;
```
###### Insights:The Engineering department receives the highest average salary, while the Customer Support department has the lowest indicating a significant pay gap across departments that may warrant review.

##### Q2:How much % employees earn more than the company-wide average salary?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/4cf0a69a4d4289d26969c8c1a45bbbc8921c7c9d/Screenshot/Q2.png)

```bash
SELECT 
      ROUND(100.0 * COUNT(*) FILTER (WHERE Salary>(SELECT round(AVG(Salary),2) AS avg_salary 
	  FROM HR_Data)) / COUNT(*), 1) AS Employee_Percent
FROM HR_Data
```
###### Insights: 448 out of 1000 employees (44.8%) earn more than the company-wide average salary.

##### Q3:Who is the highest-paid employee in each department?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/4cf0a69a4d4289d26969c8c1a45bbbc8921c7c9d/Screenshot/Q3.png)

```bash
SELECT   
     DISTINCT ON (Dept_ID) Dept_ID,Department, Full_Name, Salary
FROM 
    HR_Data
ORDER BY 
       Dept_ID, Salary DESC;
```
###### Insights: There are 8 employees who are the highest earners within their respective departments one top earner identified per department.

##### Q4:How many employees earn more than their own department's average salary?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/4cf0a69a4d4289d26969c8c1a45bbbc8921c7c9d/Screenshot/Q4.png)
```bash
WITH Dept_avg AS (
    SELECT Department, ROUND(AVG(Salary), 2) AS avg_salary
    FROM HR_Data
    GROUP BY Department
)
SELECT 
     COUNT(*) AS Employees_Above_Dept_Avg
FROM HR_Data h1
    JOIN Dept_avg d2 ON d2.Department = h1.Department
WHERE h1.Salary > d2.avg_salary;
```
###### Insights:514 employees earn more than their own department's average salary indicating a large portion of the workforce is compensated above their department's typical pay level


##### Q5:What is the average salary hike percentage for each job role, and which roles received the highest & lowest average hikes?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/4cf0a69a4d4289d26969c8c1a45bbbc8921c7c9d/Screenshot/Q5.png)
```bash
SELECT  
     Job_Role, Round(AVG(Salary_Hike_Percent),2) AS AVG_Salary_Hike 
FROM HR_Data
GROUP BY Job_Role
ORDER BY AVG_Salary_Hike DESC
```
###### Insight: IT Manager received the highest average salary hike at 9.92%, while HR Executive received the lowest at 7.42% a gap of 2.5 percentage points between the top and bottom roles.

##### Q6:What is the year-wise employee attrition trend — how many employees left each year,and what percentage of the total workforce did that represent?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/f9a13b18151b21150522f36488a496a1164f73e5/Screenshot/6.png)

```bash
SELECT Year,
       COUNT(*) FILTER (WHERE Attrition = 'Yes') AS Attrition_Count,
       COUNT(*) AS Total_Employees,
       ROUND(100.0 * COUNT(*) FILTER (WHERE Attrition = 'Yes') / COUNT(*), 2) AS Attrition_Rate_Percent
FROM HR_Data
GROUP BY Year
ORDER BY Year;
```

###### Insight: Attrition peaked in 2022 at 17.81% and dropped to its lowest in 2023 at 11.72% a significant improvement of about 6 percentage points, suggesting retention efforts may have improved during that period.

##### Q7:Which performance rating group has the highest attrition rate, and how does it compare across all rating levels?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/f9a13b18151b21150522f36488a496a1164f73e5/Screenshot/7.png)
```bash
SELECT  
      Performance_Rating,
      Count(*) AS Total_Employee,
      COUNT(*) FILTER (WHERE Attrition = 'Yes') AS Attrition_Count,
	  ROUND(100.0 * COUNT(*) FILTER (WHERE Attrition = 'Yes')/COUNT(*),1) AS Attrition_Rate_Percent
FROM HR_Data
GROUP BY Performance_Rating
ORDER BY  Attrition_Rate_Percent DESC
```
###### Insight: Attrition is highest among "Below Average" rated employees (46.2%), while "Average" rated employees have the lowest attrition rate (8.7%) — indicating lower performance is strongly linked to higher attrition.

##### Q8: At what tenure stage (years at the company) do employees tend to leave the most?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/f9a13b18151b21150522f36488a496a1164f73e5/Screenshot/8.png)
```bash
SELECT  
     Years_At_Company,
     COUNT(*) FILTER (WHERE Attrition = 'Yes') AS Attrition_Count
FROM HR_Data
GROUP BY Years_At_Company
ORDER BY  Attrition_Count DESC
```
###### Insight: Employees with 2 years of tenure have the highest attrition count (43), followed by 3 years (36) and 0 years (34)showing early tenure is the most vulnerable period for attrition.

##### Q9:Which job roles have the highest attrition, ranked from most to least ?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/f9a13b18151b21150522f36488a496a1164f73e5/Screenshot/9.png)
```bash
WITH Attrition AS (
SELECT
     Job_Role,
      COUNT(*) FILTER (WHERE Attrition = 'Yes') AS Attrition_Count
FROM HR_Data
GROUP BY Job_Role
)
SELECT Job_Role,  Attrition_Count,
dense_rank() OVER (Order by Attrition_Count desc) AS dnrnk
FROM Attrition
ORDER BY dnrnk
```
##### Insights:DevOps Engineer has the highest attrition (rank 1), followed by Sales Executive (rank 2) and Sales Manager (rank 3)

##### Q10:Which age group experiences the highest employee attrition?
![Image](https://github.com/rtrahulthapa9-byte/HR-Analytics-Dashboard-Excel/blob/f9a13b18151b21150522f36488a496a1164f73e5/Screenshot/10.png)
```bash
SELECT 
      Age_Range,
	  COUNT(*) FILTER (WHERE Attrition = 'Yes') AS Attrition_Count
FROM HR_Data
GROUP BY Age_Range
```
###### Insight: The 31–40 age group has the highest attrition, followed by 41–50, while the 20–30 age group shows the lowest attrition  suggesting mid-career employees may be more prone to leaving than younger, early-career employees.

#### Overall Takeaway
#### Attrition risk is concentrated among below-average performers, early-tenure employees, and specific job roles (DevOps, Sales), while salary distribution varies more significantly within departments than at the company-wide level — pointing to potential internal pay equity concerns alongside targeted retention needs.





