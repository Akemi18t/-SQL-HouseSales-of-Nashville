# _SQL_HouseSales-of-Nashville
This project outlines the process of cleaning data from a CSV file

# Introduction
In this project, we examine key trends and economic influences in the Nashville housing market, revealing significant changes in buyer preferences and property dynamics during that period. Several dimensions are considered in the analysis, including peak sales years, trends by construction year, bedroom configurations, popular property types, and the effect of vacant properties on sales prices.

# Tools Used 
In my thorough exploration of house sales of Nashville, I utilized a range of essential tools:

- SQL: The foundation of my analysis, enabling database querying and uncovering crucial insights. This process also involved database cleaning using SQL language.
- MicrosoftSQL: The selected database management system, well-suited for managing job posting data.
- Git & GitHub: Essential for version control and sharing my SQL scripts and analyses, ensuring collaboration and project tracking.
- Tableau: Leveraged as my visualization tool to present data with clarity, using graphs, bars, and packed bubbles for easy comprehension.

# The Analysis
### 1. Address Cleaning and Updating
I identified missing or incomplete PropertyAddress values associated with specific ParcelIDs in NashvilleHousing..

```sql
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM PorfolioProject.dbo.NashvilleHousing a
JOIN PorfolioProject.dbo.NashvilleHousing b
	 ON a.ParcelID = b.ParcelID
	 AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM PorfolioProject.dbo.NashvilleHousing a
JOIN PorfolioProject.dbo.NashvilleHousing b
	 ON a.ParcelID = b.ParcelID
	 AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL
```

### 2. Address Splitting
```sql
SELECT PropertyAddress
FROM PorfolioProject.dbo.NashvilleHousing

SELECT
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) AS Address,
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) AS Address
FROM PorfolioProject.dbo.NashvilleHousing

ALTER TABLE NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

ALTER TABLE NashvilleHousing
ADD PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))
```

### 3. Owner Address Splitting
```sql
SELECT OwnerAddress
FROM PorfolioProject.dbo.NashvilleHousing

SELECT
PARSENAME(REPLACE(OwnerAddress,',','.'),3)
,PARSENAME(REPLACE(OwnerAddress,',','.'),2)
,PARSENAME(REPLACE(OwnerAddress,',','.'),1)
FROM PorfolioProject.dbo.NashvilleHousing

ALTER TABLE NashvilleHousing
ADD OwnerSplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'),3)

ALTER TABLE NashvilleHousing
ADD OwnerSplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'),2)

ALTER TABLE NashvilleHousing
ADD OwnerSplitState NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'),1)

```

### 4. Updating 'SoldAsVacant' Field
```sql
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM PorfolioProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2

SELECT SoldAsVacant
, CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	   WHEN SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
FROM PorfolioProject.dbo.NashvilleHousing
--Update 
UPDATE NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'YES'
       WHEN SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
```

### 5. Removing Duplicates and NULL Values
```sql
WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
		         PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY 
				     UniqueID
					 ) row_num

FROM PorfolioProject.dbo.NashvilleHousing
--ORDER BY ParcelID
)

SELECT *
FROM RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress
```
### 
```sql
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM PorfolioProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2

SELECT SoldAsVacant
, CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	   WHEN SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
FROM PorfolioProject.dbo.NashvilleHousing
--Update 
UPDATE NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'YES'
       WHEN SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
SELECT *
FROM PorfolioProject.dbo.NashvilleHousing

ALTER TABLE PorfolioProject.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress

ALTER TABLE PorfolioProject.dbo.NashvilleHousing
DROP COLUMN SaleDate

SELECT *
FROM PorfolioProject.dbo.NashvilleHousing
WHERE OwnerName IS NOT NULL
  AND Acreage IS NOT NULL
  AND LandValue IS NOT NULL
  AND BuildingValue IS NOT NULL
  AND TotalValue IS NOT NULL
  AND YearBuilt IS NOT NULL
  AND Bedrooms IS NOT NULL
  AND FullBath IS NOT NULL
  AND HalfBath IS NOT NULL
  AND OwnerSplitAddress IS NOT NULL
  AND OwnerSplitCity IS NOT NULL
  AND OwnerSplitState IS NOT NULL

```
# Conclution (Visualization Graph is Provided in my Tableau Account)
[Link to My Tableau Visualization](https://public.tableau.com/app/profile/akemi.taira.vasquez/viz/HouseSalesofNashville2013-2019/Dashboard1?publish=yes)


***Yearly Peak Sales in the Housing Market (2013-2019)**:
The analysis of Yearly Peak Sales in the House Market (2013-2019) reveals significant trends and economic influences. In 2016, the market peaked at an average sales price of $304,139, possibly buoyed by improved economic conditions under President Obama's administration and policies like the Dodd-Frank Act aimed at housing affordability and recovery. Conversely, 2013 had the second-highest average sales price at $246,343, reflecting recovery post-2008 financial crisis with stricter lending practices and economic uncertainty.


***Average Sales Price by Year Built**:
The analysis of Average Sales Price by Year Built in Nashville (2013-2019) shows significant shifts in housing preferences. Starting in the 2000s, there was a notable rise in newly constructed homes, likely driven by Nashville's rapid growth and demographic changes. Despite this trend, houses built in the 1830s recorded the highest average sales, suggesting enduring appeal due to historical or architectural significance. These insights inform real estate strategies, balancing the allure of historical properties with modern constructions.

***Average Sales Price by Number of Bedrooms**:
The analysis of Average Sales Price by Number of Bedrooms reveals a clear trend where higher room numbers correlate with increased prices, reflecting larger and more expensive properties. 

***Most Popular Types of Properties Sold**:
The Most Popular Types of Properties Sold highlight that single-family homes and daycare facilities were the most purchased property types in Nashville.
This trend may indicate a growing population in the area, driving demand for residential and childcare facilities. 

**Property Sold as Vacant in Nashville(YES/NO)**:
The average price of a vacant home differs by $189,822, while a non-vacant home sells for $308,597 on average. In light of the disparity, it suggests that vacant status could influence property values, possibly reflecting market conditions or buyer perceptions. Investigating the factors behind this pricing difference could provide valuable insights for real estate decision-making and investing.


[Link to My Tableau Visualization](https://public.tableau.com/app/profile/akemi.taira.vasquez/viz/HouseSalesofNashville2013-2019/Dashboard1?publish=yes)

Thank You!
