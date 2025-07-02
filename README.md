# IBM_HR_Attrition

## Problem Statement
Company X is losing employees at an alarming rate, and want to identify key reasons contributing to attrition.

## Objective
To identify the most contributing factors to employees' desire to leave the company. As a secondary objective, Company X wants to identify if trends differ for different groups (department, role, gender).
    
## Dataset
- Source: [Kaggle - IBM HR Analytics Employee Attrition & Performance] (https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset/data)
- Records: 1471 | Columns: 35
- Features: Age, Daily Rate, Education, Job Level, Attrition

## Tools
- Excel
- BigQuery
- R, RStudio, R Markdown
- Tableau
- GitHub

## Approach

### 1. Data Cleaning & Preprocessing

The raw dataset had records of 1471 employees, before they could be analyzed however, the dataset was checked in Excel to ensure no missing values or duplicates.

#### Tasks Performed

Created a version 2 of the original dataset in Excel in case the original was needed:

**Duplicates**
No duplicates were located.

**Missing Values**
No missing values were identified.

**Removed Irrelevant Columns**
- Columns 'Employee Count', 'Over18', 'StandardHours' had the same values for all rows and were thus deleted.

**Data Preparation for Importing to SQL**
- Cells containing 'Research & Development' were replaced with 'Research and Development' to avoid issues in SQL.

With that complete, the dataset is imported to SQL

### Data Analysis

It is important to note that in this dataset the column Attrition is a Boolean value, where 'true' indicates that the employee has already served notice.

First of all, it was important to calculate the attrition rate:

```sql
SELECT
  total_employees,
  attrition_employees,
  (attrition_employees / total_employees)*100 AS attrition_rate
FROM(
  SELECT
  COUNT(*) AS total_employees,
  COUNTIF(Attrition = TRUE) AS attrition_employees,
  FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
) AS attrition_stats
```

**Total employees: 1470**  
**Attrition employees: 237**  
**Attrition rate: 16.12%**  

With that in mind, it was valuable to check whether there would be marked differences between those leaving and not leaving when it comes to the most obvious metrics. Though we need to understand that job satisfaction and attrition can often have the same causes.

```sql
SELECT
  Attrition,
  ROUND(AVG(JobSatisfaction),2)
FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
GROUP BY
  Attrition
```

**Average job satisfaction for those staying: 2.78**  
**Average job satisfaction for those leaving: 2.47**

An average difference in job satisfaction of 0.31 is significant but not large. I also wanted to check if the years since last promotion influenced whether employees were leaving, I excluded those with the highest job level (5) as that indicates no room for promotion.

```sql
SELECT
  ROUND(AVG(CASE WHEN Attrition = FALSE THEN YearsSinceLastPromotion ELSE NULL END),2) AS no_attrition_employees_avg,
  ROUND(AVG(CASE WHEN Attrition = TRUE THEN YearsSinceLastPromotion ELSE NULL END),2) AS attrition_employees_avg
FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
WHERE
  JobLevel != 5
```

**Average years since last promotion for those staying: 2.12**
**Average years since last promotion for those leaving: 1.83**

This result indicates that lack of career growth does not significantly impact attrition rates. Those more settled in their position are more likely to stay.

Before delving further into potential reasons why, I wanted to see if there were particular groups that were disproportionally giving notices. Starting with the 3 departments of this company.

```sql
SELECT
  Department,
  total_employees,
  attrition_employees,
  (attrition_employees / total_employees)*100 AS attrition_rate
FROM(
  SELECT
  Department,
  COUNT(*) AS total_employees,
  COUNTIF(Attrition = TRUE) AS attrition_employees,
  FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
  GROUP BY
    Department
) AS attrition_stats
```

**Research & Development: 13.84%**
**Human Resources: 19.05%**
**Sales: 20.64%**

Another issue that can arise in specific work environments are cultural, and I wanted to thus check if age or gender made a marked difference in attrition rates.

```sql
SELECT
  Gender,
  ROUND((attrition_employees / total_employees)*100,2)
FROM
  (
  SELECT
  Gender,
  COUNT(*) AS total_employees,
  COUNTIF(Attrition = true) AS attrition_employees
FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
  GROUP BY
    Gender
  )
```

**Male Attrition Rate: 17.01%**
**Female Attrition Rate: 14.8%**

Neither gender deviates from the attrition rate of all employees in this case. Now the same for age groups - although somewhat arbitrary I beleive the age groups defined are significant in terms of career phases.

```sql
SELECT
  age_group,
  ROUND((attrition_employees / total_employees)*100,2)
FROM(
  SELECT 
  CASE 
    WHEN age < 30 THEN 'Under 30'
    WHEN age BETWEEN 30 AND 39 THEN '30–39'
    WHEN age BETWEEN 40 AND 49 THEN '40–49'
    ELSE '50 and above'
  END AS age_group,
  COUNT(*) AS total_employees,
  COUNTIF(Attrition = true) AS attrition_employees
  FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
  GROUP BY age_group
)
```

**Under 30 attrition rate: 27.91%**
**Between 30-39 attrition rate: 14.31%**
**Between 40-49 attrition rate: 9.74%**
**50 and above attrition rate: 13.29%**

So far this is the most significant finding, and although attrition rates tend to be higher for younger employees, this deviation is much higher then a company can normally expect. To compare, the below is the attrition rate without these employees:

```sql
SELECT
  age_group,
  ROUND((attrition_employees / total_employees)*100,2)
FROM(
  SELECT 
  CASE 
    WHEN age < 30 THEN 'Under 30'
    WHEN age BETWEEN 30 AND 99 THEN 'Above 30'
  END AS age_group,
  COUNT(*) AS total_employees,
  COUNTIF(Attrition = true) AS attrition_employees
  FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
  GROUP BY age_group
)
```

**Attrition rate for under 30s: 12.76%**

## Key Insights

## Recommendations

## Files & Navigation

## View the Full Report

## Contact
