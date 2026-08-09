# Layoffs Data cleaning - SQL

## 🛠 Tools Used :

- SQL (aggregation, window functions, CTE)

## 📈 Project Overview 

This project cleans and standardizes a 2362 row layoffs dataset exported from kaggle. The raw data contained duplicates, null and blank values, and inconsistent formats — the goal was to produce a reliable, well-structured dataset ready for data exploration.

## 🔍 Data Issues Identified

- Duplicates rows 
- Blank values in the column of industry
- Inconsistent date formats (DD/MM/YYYY vs YYYY-MM-DD)
- Extra whitespace
- Inconsistent values (e.g. conflicting entries for the same field)
- Special characters and encoding issues (e.g. symbols)

## ⚙️ Methodology

- Duplicate removal — ROW_NUMBER() + PARTITION BY on key fields (i.e. the column(s) used to identify duplicate records, company, location, industry, total_laid_off, percentage_laid_off, `date`) then deleted rows where row_num > 1
- Text standardization — TRIM()
- Inconsistent value correction —  UPDATE + SET + WHERE
- Special character cleanup — TRIM(TRAILING...)
- Date standardization — STR_TO_DATE() with explicit format and modify to the date type
- Missing value handling  —  self-joins to fill gaps in one field (e.g. industry) using another record that matches on a shared field (e.g. same company)
- Removal of unusable rows — deleting records where all key metrics are missing (verified with a SELECT before running DELETE)

## 💻 Query Examples

Identifying duplicates using ROW_NUMBER() on all relevant columns and Deleting the duplicate rows identified above (row_num > 1) :

```sql
WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`,
stage, country, funds_raised_millions) AS row_num
FROM layoffs_copie
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;

CREATE TABLE `layoffs_copie2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL, 
  `row_num`  INT 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


INSERT INTO layoffs_copie2
SELECT * ,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`,
stage, country, funds_raised_millions) AS row_num
FROM layoffs_copie;


SELECT *
FROM layoffs_copie2
WHERE row_num > 1;


DELETE
FROM layoffs_copie2
WHERE row_num > 1;
```

Standardizing Data and update the table layoffs_copie2

```sql

SELECT company, TRIM(company)
FROM layoffs_copie2;


UPDATE layoffs_copie2
SET company = TRIM(company);
```

Inconsistent values correction

```sql
SELECT *
FROM layoffs_copie2
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_copie2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

Special character cleanup and update the table

```sql

SELECT *
FROM layoffs_copie2
WHERE country LIKE 'United States%'
ORDER BY 1;

SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_copie2
ORDER BY 1;


UPDATE layoffs_copie2
SET  country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```
