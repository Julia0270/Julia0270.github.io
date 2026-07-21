# Add Shelf Life and Product Category to the `SKUattributes` Table

## Purpose

This guide explains how to populate the `SKUattributes` table with product categories and shelf life attributes so that products are assigned to the correct purchase orders.

> **Note**
>
> This guide is provided as an example. All data, table names, and SQL queries used below are fictional.

## Prerequisites

Before you begin, make sure that:

- you have received the product list from the Product Manager;
- the required SKUs exist in the `ExpirationDates` table;
- you have permission to modify database tables.

## Tables

### `SKUattributes`

| Column | Description | Data Type |
| :--- | :--- | :--- |
| **SKUid** | Product ID | INT |
| **ShelfLifeAttribute** | Product shelf life attribute | VARCHAR |
| **Category** | Product category | VARCHAR |

### `ExpirationDates`

| Column | Description | Data Type |
| :--- | :--- | :--- |
| **SKUid** | Product ID | INT |
| **ShelfLife** | Product shelf life (days) | INT |

## Procedure

### 1. Receive the product list

Request the product list from the Product Manager.

The Product Manager provides an Excel file containing the following columns:

- **SKU**
- **Category**

### 2. Insert product categories into the `SKUattributes` table

Insert each **SKUid** together with its corresponding **Category**.

```sql
INSERT INTO SKUattributes (SKUid, Category)
VALUES
    (SKUid, 'dairy'),
    (SKUid, 'snacks');
```

### 3. Create a temporary table

Create a temporary table with any name (for example, `#t1`).

Populate the table with the following values from the `ExpirationDates` table:

- **SKUid**
- **ShelfLife**

```sql
-- Drop the temporary table if it already exists
DROP TABLE IF EXISTS #t1;

-- Create and populate the temporary table
SELECT
    ed.SKUid,
    ed.ShelfLife
INTO #t1
FROM ExpirationDates ed
WHERE ed.SKUid IN ();
```

### 4. Assign shelf life attributes

Classify products according to their shelf life:

- **Short-Shelf Life** — 7 days or less
- **Long-Shelf Life** — more than 7 days

Update the `ShelfLifeAttribute` column in the `SKUattributes` table.

```sql
-- Update shelf life attributes
UPDATE sa
SET sa.ShelfLifeAttribute =
    CASE
        WHEN t1.ShelfLife <= 7 THEN 'Short-Shelf Life'
        ELSE 'Long-Shelf Life'
    END
FROM SKUattributes sa
JOIN #t1 t1
    ON sa.SKUid = t1.SKUid;
```

### 5. Remove the temporary table

After the data has been transferred to the `SKUattributes` table, the temporary table is no longer required.

Delete it.

```sql
-- Drop the temporary table
DROP TABLE IF EXISTS #t1;
```

## Result

The `SKUattributes` table now contains:

- the product category;
- the shelf life attribute.

Products can now be assigned to the appropriate purchase orders based on these attributes.

[← Back to Portfolio](https://julia0270.github.io/portfolio.html)


