# SQL-Cleaning-World-Layoff
This data cleaning project use world-layoffs dataset. Project involves cleaning a dataset related to layoff statistics of businesses operating between 2020 and 2023. The cleaning process includes steps such as handling NULL values, removing duplicates, standardizing data formats, and applying statistical methods to improve data quality.


    Select * 
    From CleaningProject..layoffs

-- 1. Remove Duplicate
-- 2. Standardize Value
-- 3. Null Values or Blank Values
-- 4. Remove any columns

-- Create staging data table for backup or original references

    Select *
    Into CleaningProject..layoffs_staging
    From CleaningProject..layoffs

    Select *
    From layoffs_staging

-- 1. Working with duplicate row
	-- This query use to get the row number to count the number of occurrences of rows with the same value
 
		Select *,
		Row_Number() Over(
		Partition by company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions  Order by company) as row_num
		From layoffs_staging
	-- Copy query above and insert into new staging table with row_num column

	-- Create a new table for no duplicate row
		USE [CleaningProject]
		GO
		CREATE TABLE [dbo].[layoffs_staging2](
			[company] [nvarchar](255) NULL,
			[location] [nvarchar](255) NULL,
			[industry] [nvarchar](255) NULL,
			[total_laid_off] [float] NULL,
			[percentage_laid_off] [nvarchar](255) NULL,
			[date] [datetime] NULL,
			[stage] [nvarchar](255) NULL,
			[country] [nvarchar](255) NULL,
			[funds_raised_millions] [float] NULL,
			[row_num] [int] NULL
		) ON [PRIMARY]
		GO

	-- Insert copied query into new table like this
		INSERT INTO layoffs_staging2
		Select *,
		Row_Number() Over(
		Partition by company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions  Order by company) as row_num
		From layoffs_staging

		SELECT *
		FROM layoffs_staging2
		WHERE row_num > 1

	-- Delete duplicate row
		DELETE FROM layoffs_staging2
		WHERE row_num > 1

-- 2. Standardizing Data
	-- 2.1. Trimming
 
		SELECT company, TRIM(company)
		FROM layoffs_staging2

		UPDATE layoffs_staging2
		SET company = TRIM(company)

	-- 2.2. Replace Inconsistent industry column
 
		SELECT DISTINCT industry
		FROM layoffs_staging2
	  
		SELECT *
		FROM layoffs_staging2
		WHERE industry LIKE 'Crypto%'

		UPDATE layoffs_staging2
		SET industry = 'Crypto'
		WHERE industry LIKE 'Crypto%'

	-- 2.3. Replace Inconsistent location column
 
		SELECT DISTINCT location
		FROM layoffs_staging2
		ORDER BY 1
  
		SELECT *
		FROM layoffs_staging2
		WHERE 
			location LIKE '%seldorf' OR
			location LIKE '%lorian%' OR
			location LIKE 'malm%' OR
			location LIKE 'others'

		UPDATE layoffs_staging2
		SET location = 
			CASE 
				WHEN location LIKE '%seldorf' THEN 'Dusseldorf' 
				WHEN location LIKE '%lorian%' THEN 'Florianapolis'
				WHEN location LIKE 'malm%' THEN 'Malmo'
				WHEN location LIKE 'Non%' THEN 'Others'
				ELSE location 
			END
		WHERE 
			location LIKE '%seldorf' OR
			location LIKE '%lorian%' OR
			location LIKE 'malm%' OR
			location LIKE 'Non%'

	-- 2.4. Replace inconsistent 2 => remove messy dot and space from country name
 
		SELECT DISTINCT country
		FROM layoffs_staging2
		ORDER BY 1

		SELECT country, TRIM(TRAILING '.' FROM country)
		FROM layoffs_staging2
		ORDER BY 1

		UPDATE layoffs_staging2
		SET country = TRIM(TRAILING '.' FROM country)
		WHERE country LIKE 'United States%'

	-- 2.5. Convert date column to timeseries
		SELECT *
		FROM layoffs_staging2

		ALTER TABLE layoffs_staging2
		ALTER COLUMN date date

-- 3. Working with NULL
	-- 3.1. Populating NULL from industry column
 
		SELECT 
			T1.company, T1.industry, T2.industry
		FROM layoffs_staging2 T1
		JOIN layoffs_staging2 T2
			ON T1.company = T2.company
			AND T1.location = T2.location
		WHERE 
			T1.industry IS NULL
			AND T2.industry IS NOT NULL

	-- 3.2. Updating NULL values ​​with non-empty values ​​contained in the same company
 
		UPDATE T1
		SET T1.industry = T2.industry
		FROM layoffs_staging2 T1
			JOIN layoffs_staging2 T2 ON T1.company = T2.company
		WHERE 
			T1.industry IS NULL
			AND T2.industry IS NOT NULL 

		UPDATE layoffs_staging2
		SET industry = 'Others'
		WHERE 
			industry = ''

-- 4. Remove unnecessary column

	ALTER TABLE layoffs_staging2
	DROP COLUMN row_num
	
 -- Final Table
 
	SELECT *
	FROM layoffs_staging2
	ORDER BY date ASC



![image](https://github.com/hudzaifahjef/SQL-Cleaning-World-Layoff/assets/171403119/68ff6156-1910-40bc-bb60-0374d94dd769)
