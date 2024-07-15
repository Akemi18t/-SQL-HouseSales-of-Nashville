# _SQL_HouseSales-of-Nashville
This project outlines the process of cleaning data from a CSV file

# Introduction


# Tools Used 
In my thorough exploration of house sales of Nashville, I utilized a range of essential tools:

-**SQL: The foundation of my analysis, enabling database querying and uncovering crucial insights. This process also involved database cleaning using SQL language.
-**MicrosoftSQL: The selected database management system, well-suited for managing job posting data.
-**Git & GitHub: Essential for version control and sharing my SQL scripts and analyses, ensuring collaboration and project tracking. 
-**Tableau: Leveraged as my visualization tool to present data with clarity, using graphs, bars, and packed bubbles for easy comprehension.

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

### 5. Removing Duplicates
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

```
# Conclution

2. ***Skills for Top-Paying Jobs**:

3. ***Most In-Demand Skills**:

4. ***Skills with Higher Salaries**:

5. ***Optimal Skills for Job Market Values**:


[Link to My Tableau Visualization](https://public.tableau.com/app/profile/akemi.taira.vasquez/viz/OptimalandIn-DemandSkillsforDataAnalystsTopPayingPositionsbyCompanyandHighestSalariesintheU_S_2023/Dashboard1)

Thank You!
