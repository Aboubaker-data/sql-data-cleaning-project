# Layoffs Data cleaning - SQL

## 🛠 Tools Used :

- SQL (aggregation, window functions, CTE)

## 📈 Project Overview 

This project cleans and standardizes a 2362 row layoffs dataset exported from Github. The raw data contained duplicates, null and blank values, and inconsistent formats — the goal was to produce a reliable, well-structured dataset ready for data exploration.

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

Removing duplicates :

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
```



