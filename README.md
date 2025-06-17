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

**Total Employees: 1470**  
**Attrition Employees: 237**  
**Attrition Rate: 16.12%**  

It was good to check if there were differences in attrition rate between the 3 departments.

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

With that in mind, it was valuable to check whether there would be marked differences between those leaving and not leaving when it comes to the most obvious metrics. Though we need to understand that job satisfaction and attrition can often have the same causes.

```sql
SELECT
  Attrition,
  ROUND(AVG(JobSatisfaction),2)
FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
GROUP BY
  Attrition
```

**Average Job Satisfaction for Staying: 2.78**  
**Average Job Satisfaction for Leaving: 2.47**

I also wanted to check if the years since last promotion influenced whether employees were leaving, I excluded those with the highest job level (5) as that indicates no room for promotion.

```sql
SELECT
  ROUND(AVG(CASE WHEN Attrition = FALSE THEN YearsSinceLastPromotion ELSE NULL END),2) AS no_attrition_employees_avg,
  ROUND(AVG(CASE WHEN Attrition = TRUE THEN YearsSinceLastPromotion ELSE NULL END),2) AS attrition_employees_avg
FROM `quantum-boulder-456218-j9.hr_employees_attrition.hr_employees`
WHERE
  JobLevel != 5
```

**Average Years Since Last Promotion for Staying: 2.12**
**Average Years Since Last Promotion for Leaving: 1.83**

## Key Insights

## Recommendations

## Files & Navigation

## View the Full Report

## Contact
