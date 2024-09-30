# Quality Assurance process execution

**Goal**: Describe the QA process carried out and include the SQL queries used to execute it.

According to multiple sources, a regular QA process for SQL databases generally involves the following steps:
- Data profiling
- Data validation
- Data cleansing
- Testing
- Documenting

Part of the QA Process has been deeply incorporated into the data cleaning and transformation process, to improve the accuracy and quality of data through profiling and validation. However, standardizing/normalizing data along with the statistical data analysis (SDA) techniques hasn't been a focus due to time constraints. 

> [!NOTE]
> The sequence of "Data cleaning" follows this order of cleaning the table: 1) `all_sessions` table (table with the most columns), 2) `analytics` table (most heavy and lots of rows), `products` table, `sales_by_sku` table, `sales_report` table.

Following QA section will just provide a recap of how data quality assurance takes place and walk through some of examples of the QA queries used and why. The list of all required QA queries is much larger given exploratory analysis that took place in this project, yet due to time constraints it won't be included.

## 1. Data profiling

- Example: Write SQL queries to determine the column name, number of non-null and null values in each table

```SQL
-- Create a summary of all columns and the number of values each column in `all_sessions`
SELECT j.column_name, COUNT(j.value)
FROM all_sessions a
	CROSS JOIN LATERAL jsonb_each_text(jsonb_strip_nulls(to_jsonb(a))) AS j(column_name, value)
GROUP BY j.column_name
ORDER BY j.column_name;
```

## 2. Data validation

- Example: Write SQL queries to validate the data in the database, for example, by checking for missing or duplicate data, or data that falls outside expected ranges

```SQL	
-- View the range of units_sold in the `analytics` table by adjusting DESC/ASC: received 135 rows from -89 to 4324
SELECT DISTINCT units_sold
FROM analytics
ORDER BY units_sold DESC;

-- Investigate if there's any pattern that causes the number products sold < 0, or it's a mistake of the positive 89 value
SELECT *
FROM analytics
WHERE units_sold = -89; 
```

## 3. Data cleansing

Example: Identify any data quality issues in the database, such as missing or inconsistent data, and write SQL queries to clean the data.

```SQL
-- Convert the visitStartTime in UNIX format column to timestamp
ALTER TABLE analytics
ALTER COLUMN visitStartTime SET DATA TYPE TIMESTAMP WITH TIME ZONE
USING TIMESTAMP WITH TIME ZONE 'EPOCH' + visitStartTime * INTERVAL '1 SECOND'; 

-- Clean the leading white space in front of some productname in the `sales_report` table
UPDATE sales_report
   SET productname = REGEXP_REPLACE(productname, '(^\s+)', '');
```

## 4 & 5. Testing and Documenting 

This part is skipped due to time and resource constraints. 

## ✅ What are your risk areas? Identify and describe them.

During the first 3 steps of data profiling, data validation, and data cleansing; one of the highest risk areas encountered was with using DDL Statements to change the raw data. It did happen when I tried to DROP / ALTER a large amount of columns and rows in the `analytics` table, and had to DROP CASCADE the table and do it all over again. 

Also, data validation should ensure data is complete (i.e. no blank or null values), unique (i.e. no duplicate values), and consistent with what we expect (eg. a decimal between a certain range). That brings us back to the difficulty of determining the primary key for more complicated tables such as `all_sessions` and `analytics`. A violation of a constraint may cascade to others that call for a more cautious approach. For instance, if we set a composite key of (full_visitorid, visitid, product_sku) but the records group by these 3 columns still have duplicates that cause incompleteness in the data by false assumptions.


