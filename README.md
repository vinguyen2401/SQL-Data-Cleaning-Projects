# SQL-Data-Cleaning-Projects
## Project Overview
This project was conducted to both satisfy my curiosity and practice data analytics skills, particularly in SQL. With the surge of layoffs in tech and other sectors, I wanted to explore whether these layoffs were concentrated in specific countries or industries or spread globally.
## Dataset
The dataset, sourced from Kaggle covers global layoffs from 2020 to 2022, across 55 countries and 27 industries. It includes nine columns detailing company name, location, industry, total and percentage of layoffs, date, funding stage, country, and funds raised in millions, offering insights into global layoff trends.
## Data source :https://www.kaggle.com/datasets/swaptr/layoffs-2022
## Tool: MySQL
## Steps Involved

1. **Create Staging Table**: 
2. **Remove Duplicates**:
3. **Standardize Data**: 
4. **Handle Missing Values**: 
5. **Final Cleanup**:

## Data Cleaning
  Step 1: Create staging table and populate it
 ```sql
CREATE TABLE world_layoffs.layoffs_staging LIKE world_layoffs.layoffs;
INSERT INTO world_layoffs.layoffs_staging
SELECT * FROM world_layoffs.layoffs;
```
Step 2: Remove Duplicates: using a CTE to remove duplicate and and applying the ROW_NUMBER() function.
```sql
WITH CTE_duplicates AS (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
           ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
    FROM world_layoffs.layoffs_staging
)
DELETE FROM world_layoffs.layoffs_staging
WHERE (company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) IN (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    FROM CTE_duplicates
    WHERE row_num > 1
);
```
Step 3: Standardize Data: In this step, I clean the data by standardizing industry values, trimming trailing punctuation in country values, and converting the date column to the correct format.
```sql 
-- Standardize nulls in industry
UPDATE world_layoffs.layoffs_staging
SET industry = NULL
WHERE industry = '';

-- Populate missing industry values using non-null data
UPDATE world_layoffs.layoffs_staging t1
JOIN world_layoffs.layoffs_staging t2
  ON t1.company = t2.company AND t1.industry IS NULL AND t2.industry IS NOT NULL
SET t1.industry = t2.industry;

-- Standardize 'Crypto' variations
UPDATE world_layoffs.layoffs_staging
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- Clean up country values (remove trailing periods)
UPDATE world_layoffs.layoffs_staging
SET country = TRIM(TRAILING '.' FROM country);

-- Convert and standardize date column
UPDATE world_layoffs.layoffs_staging
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE world_layoffs.layoffs_staging
MODIFY COLUMN `date` DATE;
```
Step 4: Handle Missing Values
``` sql
-- Remove rows where both 'total_laid_off' and 'percentage_laid_off' are null
DELETE FROM world_layoffs.layoffs_staging
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;
```
Step 5: Final Cleanup
```sql
-- Drop the 'row_num' column if it was created during deduplication
ALTER TABLE world_layoffs.layoffs_staging
DROP COLUMN IF EXISTS row_num;

-- Final check on the cleaned dataset
SELECT * FROM world_layoffs.layoffs_staging;
```
## Findings
### 1. **Industries Most Affected by Layoffs**
- **Tech and Consumer Sectors:** The Tech sector, especially post-IPO companies, faced considerable layoffs due to operational restructuring and a push to achieve profitability. Despite having raised significant funds, these companies were forced to downsize to manage investor expectations and operational efficiency.
- The **Consumer sector** also saw substantial layoffs due to the shift in consumer behavior toward online shopping, along with supply chain disruptions and rising operational costs caused by the pandemic.

### 2. **Geographical Impact**
- **United States:** Cities in the United States, particularly those with a high concentration of startups and businesses, were the most affected by layoffs. This reflects the global economic challenges that hit major business hubs hardest, leading to widespread job cuts.

### 3. **Primary Causes of Layoffs**
- **Economic, Market, and Operational Factors:** Most layoffs were driven by economic pressures, market demand shifts, and operational restructuring. Companies reacted to market volatility by reducing headcounts to maintain profitability, especially during the economic downturn and uncertain recovery periods.

### Conclusion:
The cleaned dataset reflects global layoff trends across industries and regions, providing valuable insights for businesses and policymakers. Understanding these trends can help in developing strategies to support affected industries, mitigate future risks, and protect jobs in future economic challenges.
