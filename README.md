# 🌳 Global Forest Coverage Analysis - SQL Project  

## 📌 Overview  
This **SQL-based project** provides a **detailed analysis of global forest coverage trends** over the years **1990 to 2016**. The project examines:  
✅ **Total global forest area loss** over time  
✅ **Forest area as a percentage of total land area**  
✅ **Regions with the highest and lowest forest coverage**  
✅ **Country-level changes in forest area**  
✅ **Forest area rankings by income group**  

Using **SQL queries**, this analysis enables better understanding of **deforestation trends, environmental sustainability, and global efforts to preserve forest resources**.  

---

## 📂 Dataset Details  
This project uses **three datasets** related to forest area and land distribution:  

1. **forest_area.csv**  
   - Contains **annual forest area data** for each country.  
   - **Key columns:** `country_code`, `country_name`, `year`, `forest_area_sqkm`  

2. **land_area.csv**  
   - Provides **total land area** for each country.  
   - **Key columns:** `country_code`, `total_area_sq_mi` (converted to `total_area_sqkm`).  

3. **regions.csv**  
   - Contains **region and income classification** for each country.  
   - **Key columns:** `country_code`, `region`, `income_group`  

---

## 🏗️ Data Processing Steps  
### ✅ **Creating a View for Forestation Analysis**  
A **view named `forestation`** was created by joining the three tables, allowing easier analysis of forest coverage across different countries and regions.  

```sql
CREATE VIEW forestation AS
SELECT
    fa.country_code AS CountryCode,
    fa.country_name AS CountryName,
    fa.year AS Year,
    fa.forest_area_sqkm AS ForestAreaSqKm,
    la.total_area_sq_mi AS TotalAreaSqMi,
    r.region AS Region,
    r.income_group AS IncomeGroup,
    ROUND((fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59)) * 100, 2) AS ForestAreaPercentage
FROM forest_area fa
JOIN land_area la ON fa.country_code = la.country_code AND fa.year = la.year
JOIN regions r ON fa.country_code = r.country_code;
```

### ✅ **Updating the Land Area Table**  
The `land_area` table was modified to **convert total land area from square miles to square kilometers** using:  
```sql
ALTER TABLE land_area
ADD total_area_sqkm FLOAT;

UPDATE land_area 
SET total_area_sqkm = total_area_sq_mi * 2.59;
```

---

## 🔍 Key Insights & Trends  

### 1️⃣ **Global Forest Area Loss (1990-2016)**  
✅ **Total Forest Area in 1990:** 🌳 **41282694.90 sq km**  
✅ **Total Forest Area in 2016:** 🌳 **39958245.90 sq km**  
✅ **Deforestation Loss:** **1324449.00 sq km**  
✅ **Deforestation Rate:** **3.21% decrease from 1990 to 2016**  

```sql
WITH Totals AS (
    SELECT year, SUM(forest_area_sqkm) AS total_forest_area
    FROM forest_area
    WHERE year IN (1990, 2016)
    AND country_name = 'World'
    GROUP BY year
)
SELECT 
    CAST((SELECT total_forest_area FROM Totals WHERE year = 1990) AS DECIMAL(18,2)) AS total_forest_1990,
    CAST((SELECT total_forest_area FROM Totals WHERE year = 2016) AS DECIMAL(18,2)) AS total_forest_2016,
    CAST(((SELECT total_forest_area FROM Totals WHERE year = 1990) - 
          (SELECT total_forest_area FROM Totals WHERE year = 2016)) AS DECIMAL(18,2)) AS forest_loss,
    ROUND((( (SELECT total_forest_area FROM Totals WHERE year = 1990) - 
          (SELECT total_forest_area FROM Totals WHERE year = 2016)) * 100 / 
          (SELECT total_forest_area FROM Totals WHERE year = 1990)), 2) AS loss_percent;
```

🔹 **Findings:**  
- The world lost **a significant amount of forest area** between **1990 and 2016**.  
- The percentage decline indicates **a growing concern about deforestation** and its impact on the environment.  

---

### 2️⃣ **Regions with the Highest & Lowest Forest Coverage**  
✅ **Highest Forest Coverage in 1990:** 🌍 **Region Latin America & Caribbean - 51.03% forest area**  
✅ **Lowest Forest Coverage in 1990:** 🌍 **Region Middle East & North Africa - 1.78% forest area**  
✅ **Highest Forest Coverage in 2016:** 🌍 **Region Latin America & Caribbean - 46.16% forest area**  
✅ **Lowest Forest Coverage in 2016:** 🌍 **Region Middle East & North Africa - 2.07% forest area**  

```sql
SELECT TOP 1 r.region, 
       ROUND((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sqkm)) * 100, 2) AS pct_forest_area
FROM forest_area fa
JOIN land_area la ON fa.country_code = la.country_code AND fa.year = la.year
JOIN regions r ON fa.country_code = r.country_code
WHERE fa.year = 1990
GROUP BY r.region
ORDER BY pct_forest_area DESC;
```

🔹 **Findings:**  
- **Regions with high forest area (e.g., South America) show better conservation efforts.**  
- **Desert regions or areas with rapid urbanization show the lowest forest coverage.**  
- **Deforestation rates vary significantly across regions, requiring targeted policies.**  

---

### 3️⃣ **Country-Level Analysis**  
✅ **Top 5 Countries with the Most Forest Loss (1990-2016):**  
| Rank | Country | Region | Loss in Sq Km |  
|------|---------|--------|---------------|  
| 1    | Brazil  | South America | -541510 |  
| 2    | Indonesia | Asia | -282193.98 |  
| 3    | Myanmar | Asia | -107234 |  
| 4    | Nigeria | Sub-Saharan Africa | -106506|
| 5    | Tanzania | Sub-Saharan Africa | -102320

```sql
SELECT TOP 5 WITH TIES 
    f1.country_code AS CountryCode, 
    f1.country_name AS CountryName, 
    r.region,
    ROUND((f1.forest_area_sqkm - f0.forest_area_sqkm), 2) AS [Change in Forest Area in SqKm]
FROM forest_area AS f1
JOIN forest_area AS f0 
    ON f1.country_code = f0.country_code 
    AND f1.year = 2016 
    AND f0.year = 1990
JOIN regions r 
    ON f1.country_code = r.country_code
ORDER BY [Change in Forest Area in SqKm] ASC;
```

🔹 **Findings:**  
- **Brazil and Indonesia lead in deforestation**, mainly due to agriculture and logging.  
- **Africa and Asia are experiencing rapid deforestation**, highlighting the need for stricter policies.  

---

### 4️⃣ **Forest Area by Income Group**  
✅ **Top 3 Countries with the Largest Forest Area in 2016 (by Income Group)**  

```sql
WITH IncomeForest AS (
    SELECT 
        r.income_group,
        fa.country_name,
        fa.forest_area_sqkm
    FROM forest_area fa
    JOIN regions r ON fa.country_code = r.country_code
    WHERE fa.year = 2016 
    AND fa.forest_area_sqkm <> 0
)
SELECT 
    income_group,
    country_name,
    forest_area_sqkm,
    RANK() OVER (PARTITION BY income_group ORDER BY forest_area_sqkm DESC) AS forest_rank
FROM IncomeForest;
```

🔹 **Findings:**  
- **High-income countries tend to have better forest management policies.**  
- **Low-income countries face rapid deforestation due to agriculture & land expansion.**  

---

## 🚀 Future Improvements  
🔹 **Integrate population & economic growth data** for deeper insights.  
🔹 **Analyze deforestation impact on CO₂ emissions** using climate datasets.  
🔹 **Use AI/ML models** for **predicting future forest coverage trends.**  

---

## 🤝 Connect with Me  
📧 **Email:** eden.vietnguyen@gmail.com  
🔗 **LinkedIn:** [www.linkedin.com/in/eden-nguyen](https://www.linkedin.com/in/eden-nguyen)  
🌐 **Portfolio Website:** [eden-nguyen.vercel.app](https://eden-nguyen.vercel.app/)  

