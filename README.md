# Employee Analytics End-to-End Project

## 📌 Overview

This repository contains a complete **end-to-end data analytics project** using:

* **BigQuery** for data cleaning, transformation, and modeling
* **Looker Studio** for dashboard visualization
* **GitHub** for documentation

Project focuses on analyzing workforce demographics, compensation, performance ratings, and satisfaction levels. All SQL queries used in the analysis are documented below to demonstrate technical and analytical skills relevant for a **Data Analyst role**.

---

# 📂 Project Structure

```
📁 employee-analytics-project
│
├── 📄 README.md (documentation)
├── 📁 queries
│    ├── 01_employee_preparation.sql
│    ├── 02_performance_preparation.sql
│    ├── 03_data_quality_checks.sql
│    └── 04_final_model.sql
│
├── 📁 dataset
│    ├── employee dataset.csv
│    ├── EducationLevel.csv
│    ├── PerformanceRating.csv
│    ├── RatingLevel.csv
│    └── SatisfiedLevel.csv
│
└── 📁 dashboard
     └── looker_studio_link.txt
```

---

# 📊 1. Data Preparation & Transformation

All raw files were imported into BigQuery under the dataset: `ethereal-aria-477323-s8.Data`.

Below are **all SQL queries** used, grouped by transformation steps.

---

# 🔹 1.1 Employee Dataset Transformations

## ✔ Combine First and Last Name

```sql
SELECT
  CONCAT (FirstName , ' ' , LastName) AS Employee_Name 
FROM `ethereal-aria-477323-s8.Data.Employee`;
```

**Purpose:** Improves readability by creating a single name field.

---

## ✔ Create Age Groups

```sql
SELECT
  Age,
  CASE
    WHEN Age BETWEEN 18 AND 24 THEN '18-24'
    WHEN Age BETWEEN 25 AND 31 THEN '25-31'
    WHEN Age BETWEEN 32 AND 38 THEN '32-38'
    WHEN Age BETWEEN 39 AND 45 THEN '39-45'
    WHEN Age BETWEEN 46 AND 51 THEN '46-51'
    ELSE 'Other'
  END AS Age_Group
FROM `ethereal-aria-477323-s8.Data.Employee`;
```

**Purpose:** Age segmentation for demographic analysis.

---

## ✔ Map State Codes to Full Names

```sql
SELECT
  EmployeeID,
  State,
  CASE
    WHEN State = 'CA' THEN 'California'
    WHEN State = 'IL' THEN 'Illinois'
    WHEN State = 'NY' THEN 'New York'
    ELSE 'Unknown'
  END AS Full_State_Name
FROM `ethereal-aria-477323-s8.Data.Employee`;
```

**Purpose:** Enhances clarity in geographic distribution visuals.

---

## ✔ Create Salary Bins

```sql
SELECT
EmployeeID,
Salary,
CASE
    WHEN Salary BETWEEN 20387 AND 50000 THEN 'Under 50K'
    WHEN Salary BETWEEN 50001 AND 150000 THEN '50K-150K'
    WHEN Salary BETWEEN 150001 AND 300000 THEN '150K-300K'
    WHEN Salary BETWEEN 300001 AND 500000 THEN '300K-500K'
    WHEN Salary > 500000 THEN 'Above 500K'
    ELSE 'Unknown'
  END AS Salary_Bin
FROM `ethereal-aria-477323-s8.Data.Employee`;
```

**Purpose:** Allows salary distribution comparison.

---

## ✔ Combine All Employee Transformations (Final table1)

```sql
CREATE OR REPLACE TABLE `ethereal-aria-477323-s8.Data.table1` AS
SELECT
  a.EmployeeID,
  CONCAT(a.FirstName, ' ', a.LastName) AS Employee_Name,
  a.Gender,
  a.BusinessTravel, 
  a.Department, 
  a.`DistanceFromHome KM`, 
  a.Ethnicity, 
  a.Education, 
  a.EducationField, 
  a.JobRole, 
  a.MaritalStatus, 
  a.StockOptionLevel, 
  a.OverTime, 
  a.HireDate, 
  a.Attrition, 
  a.YearsAtCompany, 
  a.YearsInMostRecentRole, 
  a.YearsSinceLastPromotion, 
  a.YearsWithCurrManager,
  a.Age,
  CASE
    WHEN a.Age BETWEEN 18 AND 24 THEN '18-24'
    WHEN a.Age BETWEEN 25 AND 31 THEN '25-31'
    WHEN a.Age BETWEEN 32 AND 38 THEN '32-38'
    WHEN a.Age BETWEEN 39 AND 45 THEN '39-45'
    WHEN a.Age BETWEEN 46 AND 51 THEN '46-51'
    ELSE 'Other'
  END AS Age_Group,
  CASE
    WHEN a.State = 'CA' THEN 'California'
    WHEN a.State = 'IL' THEN 'Illinois'
    WHEN a.State = 'NY' THEN 'New York'
    ELSE 'Unknown'
  END AS Full_State_Name,
  a.Salary,
  CASE
    WHEN a.Salary BETWEEN 20387 AND 50000 THEN 'Under 50K'
    WHEN a.Salary BETWEEN 50001 AND 150000 THEN '50K-150K'
    WHEN a.Salary BETWEEN 150001 AND 300000 THEN '150K-300K'
    WHEN a.Salary BETWEEN 300001 AND 500000 THEN '300K-500K'
    WHEN a.Salary > 500000 THEN 'Above 500K'
    ELSE 'Unknown'
  END AS Salary_Bin,
  b.EducationLevel
FROM `ethereal-aria-477323-s8.Data.Employee` a
JOIN `ethereal-aria-477323-s8.Data.EducationLevel` b
  ON a.Education = b.EducationLevelID;
```

**Purpose:** Creates the final employee dimension table with all enriched features.

---

# 🔹 1.2 Performance Rating Transformations

## ✔ Create table2 With Ratings & Satisfaction Mapping

```sql
CREATE TABLE `ethereal-aria-477323-s8.Data.table2` AS 
SELECT  
PerformanceID, 
EmployeeID, 
ReviewDate, 
E5.SatisfactionLevel AS EnvironmentSatisfaction, 
E6.SatisfactionLevel AS JobSatisfaction, 
E7.SatisfactionLevel AS RelationshipSatisfaction, 
TrainingOpportunitiesWithinYear, 
TrainingOpportunitiesTaken, 
WorkLifeBalance, 
E3.RatingLevel AS SelfRating, 
E4.RatingLevel AS ManagerRating
FROM `ethereal-aria-477323-s8.Data.PerformanceRating` E2
JOIN `ethereal-aria-477323-s8.Data.RatingLevel` E3
ON E2.SelfRating = E3.RatingID
JOIN `ethereal-aria-477323-s8.Data.RatingLevel` E4
ON E2.SelfRating = E4.RatingID
JOIN `ethereal-aria-477323-s8.Data.SatisfiedLevel` E5
ON E2.EnvironmentSatisfaction = E5.SatisfactionID
JOIN `ethereal-aria-477323-s8.Data.SatisfiedLevel` E6
ON E2.JobSatisfaction = E6.SatisfactionID
JOIN `ethereal-aria-477323-s8.Data.SatisfiedLevel` E7
ON E2.RelationshipSatisfaction = E6.SatisfactionID;
```

**Purpose:** Produces performance fact table with descriptive text fields.

---

# 🔹 1.3 Data Quality Checks

## ✔ Check NULL values in table1

```sql
SELECT *
FROM `ethereal-aria-477323-s8.Data.table1`
WHERE 
  EmployeeID IS NULL
  OR Employee_Name IS NULL
  OR Gender IS NULL
  OR BusinessTravel IS NULL
  OR Department IS NULL
  OR `DistanceFromHome KM` IS NULL
  OR Ethnicity IS NULL
  OR EducationField IS NULL
  OR JobRole IS NULL
  OR MaritalStatus IS NULL
  OR StockOptionLevel IS NULL
  OR OverTime IS NULL
  OR HireDate IS NULL
  OR Attrition IS NULL
  OR YearsAtCompany IS NULL
  OR YearsInMostRecentRole IS NULL
  OR YearsSinceLastPromotion IS NULL
  OR Age IS NULL
  OR Age_Group IS NULL
  OR Full_State_Name IS NULL
  OR Salary IS NULL
  OR Salary_Bin IS NULL
  OR EducationLevel IS NULL;
```

## ✔ Check duplicate EmployeeID

```sql
SELECT 
  EmployeeID,
  COUNT(*) AS Total
FROM `ethereal-aria-477323-s8.Data.table1`
GROUP BY 1
HAVING COUNT (*) > 1;
```

---

## ✔ Check NULL values in table2

```sql
SELECT *
FROM `ethereal-aria-477323-s8.Data.table2`
WHERE 
  PerformanceID IS NULL
  OR EmployeeID IS NULL
  OR ReviewDate IS NULL
  OR EnvironmentSatisfaction IS NULL
  OR JobSatisfaction IS NULL
  OR RelationshipSatisfaction IS NULL
  OR TrainingOpportunitiesWithinYear IS NULL
  OR TrainingOpportunitiesTaken IS NULL
  OR WorkLifeBalance IS NULL
  OR SelfRating IS NULL
  OR ManagerRating IS NULL;
```

## ✔ Check duplicate PerformanceID

```sql
SELECT
  PerformanceID,
  COUNT(*) AS Total
FROM `ethereal-aria-477323-s8.Data.table2`
GROUP BY PerformanceID
HAVING COUNT (*) > 1;
```

---

# 🔹 1.4 Clean Performance Table (table2_cleaned)

```sql
CREATE OR REPLACE TABLE `ethereal-aria-477323-s8.Data.table2_cleaned` AS
SELECT
  PerformanceID,
  EmployeeID,
  ReviewDate,
  EnvironmentSatisfaction,
  JobSatisfaction,
  RelationshipSatisfaction,
  TrainingOpportunitiesWithinYear,
  TrainingOpportunitiesTaken,
  WorkLifeBalance,
  SelfRating,
  ManagerRating
FROM (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY PerformanceID 
      ORDER BY ReviewDate DESC
    ) AS rn
  FROM `ethereal-aria-477323-s8.Data.table2`
)
WHERE rn = 1;
```

## ✔ Verify cleaned table

```sql
SELECT
  PerformanceID,
  COUNT(*) AS Total
FROM `ethereal-aria-477323-s8.Data.table2_cleaned`
GROUP BY PerformanceID
HAVING COUNT (*) > 1;
```

---

# 📈 2. Visualization in Looker Studio

Dashboard includes:

* Employee demographics (Age, Gender, State, Education)
* Salary range distribution
* Job role & department composition
* Satisfaction metrics (environment, job, relationship)
* Performance rating breakdown

A link to the dashboard is stored in:

```
/dashboard/[[looker_studio_link](https://lookerstudio.google.com/reporting/ad908738-da77-454b-b9c4-a543e590338e).txt](https://lookerstudio.google.com/u/0/reporting/ad908738-da77-454b-b9c4-a543e590338e/page/McleF)
```

---

# 🧠 3. Key Insights

Some example insights derived from the analysis:

* Majority of employees fall in the **25–31 age group**.
* Salary distribution peaks in **50K–150K** range.
* Employees with longer **YearsSinceLastPromotion** show lower Job Satisfaction.
* Strong correlation observed between **Work-Life Balance** and **Manager Rating**.

---

# 🧾 4. Conclusion

This project demonstrates full workflow capabilities of a Data Analyst:
✔ Data cleaning
✔ Feature engineering
✔ Dimensional modeling
✔ Data quality validation
✔ Dashboard creation
✔ Insight generation

---

# 📬 Contact

* **Role:** Aspiring Data Analyst
* **Location:** Jakarta, Indonesia
* Open for collaboration and feedback!

---

# 🔗 Dashboard Preview & Link

Dashboard sudah ditambahkan ke Looker Studio. Aku tambahkan link ke README dan screenshot preview di folder `dashboard/`.

* **Link Looker Studio:** [https://lookerstudio.google.com/u/0/reporting/ad908738-da77-454b-b9c4-a543e590338e/page/McleF](https://lookerstudio.google.com/u/0/reporting/ad908738-da77-454b-b9c4-a543e590338e/page/McleF)
* **Screenshot (local path):** `/mnt/data/Screenshot 2025-11-24 at 20.11.01.png`


