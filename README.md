# Data Cleaning SQL Project - Layoffs 2022

## Overview
This project focuses on cleaning and preparing the **Layoffs 2022** dataset sourced from [Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022). The dataset contains information about global layoffs in various industries. The goal of this project is to perform data cleaning using SQL to ensure accuracy, consistency, and usability of the dataset for further analysis.

## Dataset
- Source: Kaggle
- Table Name: `world_layoffs.layoffs`
- Staging Table: `world_layoffs.layoffs_staging`

## Steps Performed
### 1. Creating a Staging Table
Before modifying the raw dataset, a staging table was created to work on the data while preserving the original dataset.
```sql
CREATE TABLE world_layoffs.layoffs_staging LIKE world_layoffs.layoffs;
INSERT INTO world_layoffs.layoffs_staging SELECT * FROM world_layoffs.layoffs;
```

### 2. Removing Duplicates
Duplicates were identified and removed based on multiple columns to ensure uniqueness.
```sql
WITH DELETE_CTE AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num
    FROM world_layoffs.layoffs_staging
)
DELETE FROM world_layoffs.layoffs_staging WHERE row_num > 1;
```

### 3. Standardizing Data
- **Handling NULL and blank values:**
```sql
UPDATE world_layoffs.layoffs_staging2 SET industry = NULL WHERE industry = '';
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2 ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;
```

- **Standardizing inconsistent values:**
```sql
UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
UPDATE layoffs_staging2 SET country = TRIM(TRAILING '.' FROM country);
```

- **Fixing date format:**
```sql
UPDATE layoffs_staging2 SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
ALTER TABLE layoffs_staging2 MODIFY COLUMN `date` DATE;
```

### 4. Handling NULL Values
NULL values in `total_laid_off`, `percentage_laid_off`, and `funds_raised_millions` were retained for accurate calculations in future analysis.

### 5. Removing Unnecessary Data
Rows where `total_laid_off` and `percentage_laid_off` were both NULL were deleted.
```sql
DELETE FROM world_layoffs.layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;
```
The temporary column `row_num` used for duplicate removal was also dropped.
```sql
ALTER TABLE layoffs_staging2 DROP COLUMN row_num;
```

## Final Output
The cleaned dataset in `layoffs_staging2` is now structured, standardized, and ready for further analysis.

## Tools & Technologies Used
- MySQL
- SQL Queries for Data Cleaning

## How to Use
1. Clone this repository.
2. Import the dataset into MySQL.
3. Run the SQL scripts step-by-step to clean the data.
4. Use the cleaned data for further analysis in Power BI, Excel, or Python.

## Author
Aditya Sharma

## License
This project is licensed under the MIT License.

