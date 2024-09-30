## Step 1: Loading csv Files into Database

The first step involves filtering and combining the datasets into a single workable format for analysis:

1. Create the `ecommerce` database and the tables with corresponding columns to our 5 datasets in PostgreSQL using pgAdmin4 and the ```CREATE TABLE`` statement.
2. Import the csv files into the table using ```COPY``` statement.
We haven't set up the constraints and the keys at this stage, because there are some messy missing data and duplicates in some of the unique identifier columns.

<details>

<summary>   Expand to view code  👉 </summary>

### 1.1. Creating first table in a database:

```SQL
CREATE TABLE products
	(
	sku VARCHAR,
	product_name VARCHAR,
	quantity INTEGER,
	stock_level INTEGER,
	restocking_lead_time DOUBLE PRECISION,
	sentiment_score DOUBLE PRECISION,
	sentiment_magnitude DOUBLE PRECISION
	);
```

#### Importing data to new table:

```SQL
COPY products
	(
	sku,
	product_name,
	quantity,
	stock_level,
	restocking_lead_time,
	sentiment_score,
	sentiment_magnitude 
	)
FROM 'C:\Users\AM\OneDrive\Desktop\LHL\WEEK 2\csv\products.csv'
DELIMITER ','
CSV HEADER;
```

#### Checking if copied properly:

```SQL
SELECT * FROM products;
```

### 1.2. Copy other tables:

#### sales_by_sku

```SQL
CREATE TABLE sales_by_sku
	(
	product_sku VARCHAR,
	total_ordered INTEGER
	);

COPY sales_by_sku
	(
	product_sku,
	total_ordered
	)
FROM 'C:\Users\AM\OneDrive\Desktop\LHL\WEEK 2\csv\sales_by_sku.csv'
DELIMITER ','
CSV HEADER;
```
```SQL
SELECT * FROM sales_by_sku;
```
#### sales_report

```SQL
CREATE TABLE sales_report
		(
		product_sku VARCHAR,
		total_ordered INTEGER,
		product_name VARCHAR,
		stock_level INTEGER,
		restocking_lead_time DOUBLE PRECISION,
		sentiment_score DOUBLE PRECISION,
		sentiment_magnitude DOUBLE PRECISION,
		ratio DOUBLE PRECISION
		);
```
```SQL
COPY sales_report
		(
		product_sku,
		total_ordered,
		product_name,
		stock_level,
		restocking_lead_time,
		sentiment_score,
		sentiment_magnitude,
		ratio 
		)
FROM 'C:\Users\AM\OneDrive\Desktop\LHL\WEEK 2\csv\sales_report.csv'
DELIMITER ','
CSV HEADER;
```
```SQL
SELECT * FROM sales_report;
```

#### analytics

```SQL
CREATE TABLE analytics
		(
		visit_number INTEGER,
	    visitid BIGINT,
	    visit_start_time BIGINT,
	    visit_date DATE,
	    full_visitorid NUMERIC(20,0),
	    userid VARCHAR(255),
	    channel_grouping VARCHAR(50),
	    social_type VARCHAR(50),
	    units_sold INTEGER,
	    page_views INTEGER,
	    time_on_site INTEGER,
	    bounces INTEGER,
	    revenue NUMERIC(12,2),
	    unit_price NUMERIC(12,2)
		);
```
```SQL
COPY analytics
		(
		visit_number,
		visitid,
		visit_start_time,
		visit_date,
		full_visitorid,
		userid,
		channel_grouping,
		social_type,
		units_sold,
		page_views,
		time_on_site,
		bounces,
		revenue,
		unit_price
		)
FROM 'C:\Users\AM\OneDrive\Desktop\LHL\WEEK 2\csv\analytics.csv'
DELIMITER ','
CSV HEADER;
```
```SQL
SELECT * FROM analytics;
```

#### all_sessions

```SQL
CREATE TABLE all_sessions
		(
		full_visitorid NUMERIC,
		channel_grouping VARCHAR,
		visit_start_time INTEGER,
		country VARCHAR,
		city VARCHAR,
		total_transaction_revenue NUMERIC,
		transactions INTEGER,
		time_on_site INTEGER,
		pageviews INTEGER,
		session_quality INTEGER,
		visit_date DATE,
		visitid BIGINT,
		visit_type VARCHAR,
		product_refund_amount NUMERIC,
		product_quantity INTEGER,
		product_price NUMERIC,
		product_revenue NUMERIC,
		product_sku VARCHAR,
		product_name2 VARCHAR,
		product_category2 VARCHAR,
		product_variant VARCHAR,
		currency CHAR(3),
		item_quantity INTEGER,
		item_revenue NUMERIC,
		transaction_revenue NUMERIC,
		transactionid VARCHAR,
		page_title VARCHAR,
		search_keyword VARCHAR,
		page_path VARCHAR,
		action_type INTEGER,
		action_step INTEGER,
		action_option VARCHAR
		);
```
```SQL
COPY all_sessions
		(
		full_visitorid,
		channel_grouping,
		visit_start_time,
		country,
		city,
		total_transaction_revenue,
		transactions,
		time_on_site,
		pageviews,
		session_quality,
		visit_date,
		visitid,
		visit_type,
		product_refund_amount,
		product_quantity,
		product_price,
		product_revenue,
		product_sku,
		product_name2,
		product_category2,
		product_variant,
		currency,
		item_quantity,
		item_revenue,
		transaction_revenue,
		transactionid,
		page_title,
		search_keyword,
		page_path,
		action_type,
		action_step,
		action_option
		)
FROM 'C:\Users\AM\OneDrive\Desktop\LHL\WEEK 2\csv\all_sessions.csv'
DELIMITER ','
CSV HEADER;
```
```SQL
SELECT * FROM all_sessions;
```

</details>

## Step 2: Data Cleaning and Transforming

The overall approach to clean the data across our 5 tables:

- Remove irrelevant, duplicated data
- Fix structural errors
- Perform type conversion
- Handle missing data

### 2.1. Cleaning and transforming the `all_sessions` table

Before deciding to drop any irrelevant or missing data, it's better to summarize how many values there are in `all_sessions`.

```SQL
SELECT 
    j.column_name, COUNT(j.value)
FROM all_sessions a
	CROSS JOIN LATERAL jsonb_each_text(jsonb_strip_nulls(to_jsonb(a))) AS j(column_name, value)
GROUP BY j.column_name
ORDER BY j.column_name;
```

The return output shows that there are missing values in some of the columns, mostly the transaction related columns such as transactionid, transaction_revenue, total_transaction_revenue, transactions. Missing values also scatter across the set of ecommerce, product columns and columns like action_option, session_quality.

### 2.2. Handling duplicates

Find full duplicates in all columns and drop them if irrelevant. The query results in 0 rows.
For the sake of saving space, I included only `all_sessions` table.

<details>

<summary>   Expand to view code  👉 </summary>

```SQL
SELECT
    (
    full_visitorid,
		channel_grouping,
		visit_start_time,
		country,
		city,
		total_transaction_revenue,
		transactions,
		time_on_site,
		pageviews,
		session_quality,
		visit_date,
		visitid,
		visit_type,
		product_refund_amount,
		product_quantity,
		product_price,
		product_revenue,
		product_sku,
		product_name2,
		product_category2,
		product_variant,
		currency,
		item_quantity,
		item_revenue,
		transaction_revenue,
		transactionid,
		page_title,
		search_keyword,
		page_path,
		action_type,
		action_step,
		action_option
    )
FROM all_sessions
GROUP BY 
	full_visitorid,
		channel_grouping,
		visit_start_time,
		country,
		city,
		total_transaction_revenue,
		transactions,
		time_on_site,
		pageviews,
		session_quality,
		visit_date,
		visitid,
		visit_type,
		product_refund_amount,
		product_quantity,
		product_price,
		product_revenue,
		product_sku,
		product_name2,
		product_category2,
		product_variant,
		currency,
		item_quantity,
		item_revenue,
		transaction_revenue,
		transactionid,
		page_title,
		search_keyword,
		page_path,
		action_type,
		action_step,
		action_option
HAVING COUNT(*) > 1;
```

</details>

### 2.3. Setting primary and foreign keys

Based on 2.2 output, we might need to pick a combination of columns as the unique identifer for the `all_sessions` table, which can be (full_visitorid, visitid, product_sku) considering the relations to other tables in the database. Within the scope of this project, we won't emphasize on setting up the primary key and other constraints for each table. Our priority would just be defining the connection with other tables.


#### A simple way to choose primary keys is just by looking the data sample and identifying the rows that seems unique, then running a following query:

``` SQL
SELECT COUNT(*), full_visitorid
FROM all_sessions
GROUP BY full_visitorid HAVING COUNT(full_visitorid) > 1
```
#### Creating PK in all_sessions was challenging - both visitid and full_visitorid turned out to not be unique.
#### Let's move on to creating them for other tables:

``` SQL
ALTER TABLE sales_report
ADD PRIMARY KEY (product_sku);

ALTER TABLE products
ADD PRIMARY KEY (product_sku);
```
####  Adding constraints:

``` SQL
ALTER TABLE sales_report
ADD CONSTRAINT fk_product_sku
FOREIGN KEY (product_sku)
REFERENCES products(product_sku);
```

> [!WARNING]
> product_sku cannot be FK in sales_by_sku, it has unique values not present in neither products nor sales_report tables

### 2.4. Handling discrepancies (names, typos, caps)

Next, the goal is to fix strange naming conventions, typos, or incorrect capitalization. We'll go through few columns in this section.

####   `product_sku` column: Its data type as the same VARCHAR since there's a mix of numeric values and characters such as '9182569' or 'GGOEGOAJ021599'.

####   `channel_grouping` column: Includes these distinct values: 'Organic Search', 'Display', 'Referral', 'Affiliates', 'Direct', 'Paid Search', '(Other)'. Changing the text format of '(Other)' to remove the parentheses is necessary and less confusing.

```SQL
-- View the unique values of channel_grouping
SELECT DISTINCT channel_grouping
FROM all_sessions;

-- Update '(Other)' to 'Other'
UPDATE all_sessions SET channelGrouping = BTRIM(channelGrouping,'()')
```

####   `time` column: The values here looks like Unix Timestamps with values ranging from 0 to 3192410. When we attempted to convert this column to timestamp without time zone, the result are dated from 1970-01-01 00:00:00 to 1970-02-06 22:46:50. We can drop the time column because it doesn't make sense when compared to the `date` column (with data from 2016-08-01 to 2017-08-01 below in the date column investigation). We also don't have any other source of information to fill in the data later on.

```SQL
-- Convert `time` column to timestamp
SELECT DISTINCT TO_TIMESTAMP(time::BIGINT) AT TIME ZONE 'UTC' AS time_converted
FROM all_sessions
ORDER BY time_converted;

-- Drop `time` column from the `all_sessions` table
ALTER TABLE all_sessions 
DROP COLUMN time;
```

 ####  `total_transaction_revenue`, `transactions`, `transaction_revenue`, `transactionid` columns: all fall within the same domain of transaction, we can analyze them together. The main issue here is a large amount of missing data. `total_transaction_revenue` and `transactions` most likely come from the same source with 81 records. There are only 9 `transactionid` and 4 `transactionRevenue` values. 

####  We may need to check if `total_transaction_revenue` and `transaction_revenue` are reflecting the same metric. The output confirms the duplication which means dropping `transaction_revenue` won't have any impact on our table.

```SQL
-- Check if `total_transaction_revenue` and `transaction_revenue` are duplicated
SELECT DISTINCT total_transaction_revenue, transaction_revenue
FROM all_sessions;

-- Drop `transaction_revenue` column
ALTER TABLE all_sessions 
DROP COLUMN transaction_revenue;
```

####  All the revenue and price column values seem to have a calculation error. The easiest way to verify this was to check the price of "Gift Card $250" item which costs 250,000,000 USD in `product_price` column. It's an indicator to reduce the amount by 1,000,000 for such monetary columns.

```SQL
-- Investigate the inflated value of `total_transaction_revenue` and other justifying columns
SELECT
	DISTINCT transactionid, 
	transactions,
	total_transaction_revenue,
	currency,
	product_price,
	product_quantity,
	product_name2
FROM all_sessions
WHERE transactionid IS NOT NULL;

-- Update new values in `total_transaction_revenue` and `product_price` by dividing by 1,000,000
UPDATE all_sessions
SET 
	total_transaction_revenue = ROUND(total_transaction_revenue/1000000, 2),
	product_rrice = ROUND(product_price/1000000, 2);
```
#### `currency` column: We'll fill in the missing values in `currency` with USD, as when grouping `country` and `currency`, countries without currency code are already tagged as USD.

```SQL
UPDATE all_sessions 
	SET currencyCode = CASE 
					WHEN currencyCode IS NULL THEN 'USD'
					ELSE currencyCode
				  END;
```

#### Drop the rest of unnecessary columns filled with missing values: item_quantity, item_revenue, search_keyword

```SQL
ALTER TABLE all_sessions 
DROP COLUMN itemQuantity,
DROP COLUMN itemRevenue,
DROP COLUMN searchKeyword;
```
> [!TIP]
> Do not delete any columns before exploring data and answering a few questions first.
> Even empty column may show some insights when combined with other tables.

### 2.5. Cleaning and transforming the `analytics` table

#### Below is a summary table of values (columns with all NULL values not included) in the `analytics` table. This step is necessary prior to any futher data cleaning.

```SQL
SELECT 
    j.column_name, COUNT(j.value)
FROM analytics b
	CROSS JOIN LATERAL jsonb_each_text(jsonb_strip_nulls(to_jsonb(b))) AS j(column_name, value)
GROUP BY j.column_name
ORDER BY j.column_name;
``` 

#### At the first glance, this table contains a lot of rows - up to 4.3 million rows. The return output shows alerts missing values most spread out in the bounces, revenue, units_sold columns. 

|     column_name       |      count        |
|-----------------------|:------------------|
|bounces		        |	474839          |
|channelgrouping        |	4301122         |
|date	                |	4301122         |  
|fullvisitorid          |	4301122         |
|pageviews              |	4301050         |
|revenue		 		|	15355           |
|socialengagementtype   |	4301122         |
|timeonsite		        |	3823657         |
|unit_price		        |	4301122         |
|units_sold             |	95147           |
|visitid                |	4301122         |
|visitnumber	        |	4301122         |
|visitstarttime         |	4301122         |

#### The `analytics` table has 14 columns in total in which the userid column is fully NULL. We might consider dropping userid column later if it isn't connected to other columns and tables.

```SQL
ALTER TABLE analytics 
DROP COLUMN userid;
```

#### Let's do some cleaning on the columns before we are able to determine the primary key candidate and remove the duplicates.

#### The `visit_start_time` and `date` columns: 

#### Next, we convert `visit_start_time` into the TIMESTAMP data type. Interestingly after the conversion, there's a date component inside the timestamp which doesn't fully match with the date column, sometimes 1 day later. This may have something to do with timezone. We'll have to drop the existing date column to be consistent, and compensate by extracting the date from `visit_start_time`. 

```SQL
-- Convert from BIGINT to TIMESTAMP WITH TIME ZONE

ALTER TABLE analytics
ALTER COLUMN visit_start_time SET DATA TYPE TIMESTAMP WITH TIME ZONE
USING TIMESTAMP WITH TIME ZONE 'EPOCH' + visit_start_time * INTERVAL '1 SECOND'; 

-- Check how many rows when date in `visit_start_time` different from `date` column

SELECT COUNT(*)
FROM analytics
WHERE visit_start_time::DATE != visit_date; -- zero
```

#### The `visitid` column: We first observe `visitid`, `visit_start_time` are identical, which means `visitid` values possibly have been copied from `visit_start_time` by mistake. We should find the appropriate values to fill in, as the column seems to be important as a unique identifer at a later stage. The common columns between `analytics` and `all_sessions` tables are (full_visitorid, visitid). Thus, together they are a decent composite key candidate for the `analytics` table.

```SQL
-- Fill in the missing `visitid` column in the `analytics` table with a lookup table CTE extracted from the `full_visitorid` from the `all_sessions` table

WITH visitid_lookup_CTE AS(
SELECT *
FROM (SELECT al.full_visitorid, al.visitid
	  FROM all_sessions al
	  GROUP BY al.full_visitorid, al.visitid) AS a
JOIN
	  (SELECT an.full_visitorid, an.visitid
	   FROM analytics an
	 GROUP BY an.full_visitorid, an.visitid) AS b
USING (full_visitorid, visitid)
)

INSERT INTO analytics(visitid)
SELECT v.visitid FROM visitid_lookup_CTE v
WHERE NOT EXISTS (SELECT * FROM analytics ana WHERE ana.visitid = v.visitid);
```

#### The units_sold column: Fix the odd -89 value by 89 units sold, as data ranges from -89 to 4324, nothing unusal except for this one.

```SQL
UPDATE analytics 
	SET units_sold = CASE 
						WHEN units_sold = -89 THEN 89
						ELSE units_sold
				  	 END;
```

#### The unit_price, revenue columns: Just like in the `all_sessions` table, those 2 columns are overinflated with 1000000 times more than the usual amount. We'll fix them with a more reasonable amount by dividing by 1000000.

```SQL
-- Fix revenue and unit_price columns' amount from the `analytics` table by dividing by 1000000
UPDATE analytics
SET 
	revenue = ROUND(revenue/1000000, 2),
	unit_price = ROUND(unit_price/1000000, 2);
```

#### Delete the full duplicate rows in the `analytics table` after cleaning all the tables - can be done with CONCAT

<details>

<summary>   Expand to view code  👉 </summary>

```SQL

-- Delete repeated rows in `analytics` table by using CONCAT

DELETE FROM analytics an1
WHERE CONCAT( visit_number, ' ',
			  visitid, ' ', 
			  visit_start_time, ' ',
			  date, ' ',
			  full_visitorid, ' ',
			  channel_grouping, ' ',
			  social_type, ' ',
			  units_sold, ' ',
			  page_views, ' ',
			  time_on_site, ' ',
			  bounces, ' ',
			  revenue, ' ',
			  unit_price)
NOT IN (
		SELECT DISTINCT CONCAT( visit_number, ' ',
			  visitid, ' ', 
			  visit_start_time, ' ',
			  date, ' ',
			  full_visitorid, ' ',
			  channel_grouping, ' ',
			  social_type, ' ',
			  units_sold, ' ',
			  page_views, ' ',
			  time_on_site, ' ',
			  bounces, ' ',
			  revenue, ' ',
			  unit_price
		FROM analytics an2
		GROUP BY CONCAT(		visit_number, ' ',
			  visitid, ' ', 
			  visit_start_time, ' ',
			  date, ' ',
			  full_visitorid, ' ',
			  channel_grouping, ' ',
			  social_type, ' ',
			  units_sold, ' ',
			  page_views, ' ',
			  time_on_site, ' ',
			  bounces, ' ',
			  revenue, ' ',
			  unit_price)
		HAVING COUNT(*) > 1
);
```
</details>

We might have not set the PK for this table yet, but the composite key using (full_visitorid, visitid) seems to be a good candidate. They can be used to find connection with the `all_sessions` table.

``` SQL
ALTER TABLE analytics
ADD CONSTRAINT PK_analytics PRIMARY KEY (full_visitorid, visitid);
```

## 2.6. Cleaning and transforming the `products` table

</details>


<summary>   Expand to view details  👉 </summary>


#### A summary of values in the `products` table:

```SQL
SELECT 
    j.column_name, COUNT(j.value)
FROM products p
	CROSS JOIN LATERAL jsonb_each_text(jsonb_strip_nulls(to_jsonb(p))) AS j(column_name, value)
GROUP BY j.column_name
ORDER BY j.column_name;
``` 

The result looks pretty clean with just a few missing values

|     column_name       |      count        |
|-----------------------|:------------------|
|name	        		|	1092         	|
|orderedquantity        |	1092         	|
|restockingleadtime     |	1092          	|
|sentimentmagnitude		|	1091        	|
|sentimentscore	 		|	1091           	|
|sku	 				|	1092        	|
|stocklevel		        |	1092         	|

</details>

## 2.7. No cleaning needed for the `sales_by_sku` table

## 2.8. Clean and transform the `sales_report` table

This table is pretty clean already with repeated columns probally drawn from the `products` and `sales_report` tables. We just need to change minor details in the `name` colum as we performed in the previous ones.

```SQL
--- Change column name from `name` to `product_name`
ALTER TABLE sales_report
RENAME COLUMN name TO product_name;

--- Remove the leading whitespace from `product_name`
UPDATE sales_report
   SET productname = REGEXP_REPLACE(product_name, '(^\s+)', '');
```

> WOW! You reached the end of data cleaning walkthrough! Kudos from a happy piglet 🐖
