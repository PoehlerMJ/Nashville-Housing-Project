/****** Script for SelectTopNRows command from SSMS  ******/
SELECT TOP (1000) [UniqueID ]
      ,[ParcelID]
      ,[LandUse]
      ,[PropertyAddress]
      ,[SaleDate]
      ,[SalePrice]
      ,[LegalReference]
      ,[SoldAsVacant]
      ,[OwnerName]
      ,[OwnerAddress]
      ,[Acreage]
      ,[TaxDistrict]
      ,[LandValue]
      ,[BuildingValue]
      ,[TotalValue]
      ,[YearBuilt]
      ,[Bedrooms]
      ,[FullBath]
      ,[HalfBath]
  FROM [NashHousing].[dbo].[NashHousing]

  --Standardize Date Format

  Select *
  FROM [NashHousing].[dbo].[NashHousing]

  SELECT SaleDateConverted, CONVERT(Date, SaleDate)
  FROM [NashHousing].[dbo].[NashHousing]

  Update [NashHousing].[dbo].[NashHousing]
  SET SaleDate = CONVERT(Date, SaleDate)

  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD SaleDateConverted Date;

   Update [NashHousing].[dbo].[NashHousing]
  SET SaleDateConverted = CONVERT(Date, SaleDate)
----------------------------------------------------------------------------------------------
  --- Populate Property Address Data

  Select *
  FROM [NashHousing].[dbo].[NashHousing]
  --WHERE PropertyAddress is null
  ORDER BY ParcelID

  --Self Join to populate

  Select T1.ParcelID, T1.PropertyAddress, T2.ParcelID, T2.PropertyAddress, ISNULL(T1.PropertyAddress, T2.PropertyAddress)
  FROM [NashHousing].[dbo].[NashHousing] T1
  JOIN [NashHousing].[dbo].[NashHousing] T2
  ON T1.ParcelID = T2.ParcelID
  AND T1.[UniqueID] <> T2.[UniqueID]
  WHERE T1.PropertyAddress is null

  UPDATE T1
  SET PropertyAddress = ISNULL(T1.PropertyAddress, T2.PropertyAddress)
  FROM [NashHousing].[dbo].[NashHousing] T1
  JOIN [NashHousing].[dbo].[NashHousing] T2
  ON T1.ParcelID = T2.ParcelID
  AND T1.[UniqueID] <> T2.[UniqueID]
  WHERE T1.PropertyAddress is null

  ----------------------------------------------------------------------------------------------
  ---Breaking out Address into Individual Columns

  Select PropertyAddress
  FROM [NashHousing].[dbo].[NashHousing]
  --WHERE PropertyAddress is null
 -- ORDER BY ParcelID

 SELECT 
 SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) AS Address,
 SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) as Address
 FROM [NashHousing].[dbo].[NashHousing]


  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD PropertySplitAddress Nvarchar(255);

   Update [NashHousing].[dbo].[NashHousing]
  SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD PropertySplitCity Nvarchar(255);

   Update [NashHousing].[dbo].[NashHousing]
  SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))


  SELECT OwnerAddress
  FROM [NashHousing].[dbo].[NashHousing]

  SELECT 
  PARSENAME(REPLACE(OwnerAddress, ',', '.') ,3)
  ,PARSENAME(REPLACE(OwnerAddress, ',', '.') ,2)
	,PARSENAME(REPLACE(OwnerAddress, ',', '.') ,1)
  FROM [NashHousing].[dbo].[NashHousing]

  
  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD OwnerSplitAddress Nvarchar(255);

   Update [NashHousing].[dbo].[NashHousing]
  SET OwnerSplitAddress =  PARSENAME(REPLACE(OwnerAddress, ',', '.') ,3)

  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD OwnerSplitCity Nvarchar(255);

   Update [NashHousing].[dbo].[NashHousing]
  SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.') ,2)

  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  ADD OwnerSplitState Nvarchar(255);

   Update [NashHousing].[dbo].[NashHousing]
  SET OwnerSplitState =  PARSENAME(REPLACE(OwnerAddress, ',', '.') ,1)

  SELECT *
  FROM [NashHousing].[dbo].[NashHousing]

  ----------------------------------------------------------------------------------------------
  --- Change Y and N to Yes and No in "Sold in Vacant" field

  SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
  FROM [NashHousing].[dbo].[NashHousing]
  GROUP BY SoldAsVacant
  ORDER BY 2

  SELECT SoldAsVacant,
  CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
  WHEN SoldAsVacant = 'N' THEN 'No'
  ELSE SoldAsVacant
  END
  FROM [NashHousing].[dbo].[NashHousing]

  UPDATE [NashHousing].[dbo].[NashHousing]
  SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
  WHEN SoldAsVacant = 'N' THEN 'No'
  ELSE SoldAsVacant
  END

  ----------------------------------------------------------------------------------------------
  --- Remove Duplicates

  WITH RowNumCTE AS(
  SELECT *,
  ROW_NUMBER() OVER(
  PARTITION BY ParcelID,
  PropertyAddress,
  SalePrice,
  SaleDate,
  LegalReference
  ORDER BY UniqueID
  ) row_num
  FROM [NashHousing].[dbo].[NashHousing]
  --ORDER BY ParcelID
  )
  DELETE
  FROM RowNumCTE
  WHERE row_num >1
  --ORDER BY PropertyAddress


----------------------------------------------------------------------------------------------
  --Delete Unused Columns

  SELECT *
  FROM [NashHousing].[dbo].[NashHousing]

  ALTER TABLE [NashHousing].[dbo].[NashHousing]
  DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress

    ALTER TABLE [NashHousing].[dbo].[NashHousing]
  DROP COLUMN SaleDate
 







