> [!NOTE]
> Before we dive in, please note: this document reflects my thought process. When coding, I often engage in lively conversations with myself, so please excuse any excessive mumbling. It’s just my brain trying to debug itself!

## 💡 Question 1: Which cities and countries have the highest level of transaction revenues on the site?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

Take a look at the whole table - is all required info available here?

``` SQL
SELECT * FROM all_sessions;
```

There are 2 columns that can be interpreted as "transaction revenue", this ambiguity can lead to wrong results. Checking the NULL values in both 'transaction_revenue' and 'total_transaction_revenue' (assuming that 'product_revenue' is a partial revenue, its not what we're looking for)

__EDIT:__ _changes were made to certain columns and described in "data cleaning" file_

``` SQL
SELECT  transaction_revenue, 
		product_sku
FROM all_sessions 
WHERE transaction_revenue IS NOT NULL;
```
Only 4 show any values, now let's check how many rows have values in total_transaction_revenue:

``` SQL
SELECT  total_transaction_revenue,
		product_sku
FROM all_sessions 
WHERE  total_transaction_revenue IS NOT NULL;
```

Adding another condition shows that previous 4 values are included in second column:

``` SQL
AND product_sku IN ('GGOENEBB078899', 'GGOEGEVJ023999', 'GGOEGBJR018199', 'GGOEGEVB070899');
```

Required fields are here (transaction_revenue, visit_type, city, country), so we can build a query: 

``` SQL
SELECT  SUM(total_transaction_revenue) AS revenue, 
		city,
		country
FROM all_sessions
WHERE   visit_type = 'PAGE' -- filter by visit_type ('PAGE')
AND total_transaction_revenue > 0 -- avoid NULLs in revenue
GROUP BY city, country -- group revenue by city
``` 

Some cities are missing, but revenue should still count towards country - COALESCE can solve this by filling missing values with countries. 
Order DESC and optionally limit to 10 to get top 10 cities and countries with highest revenues:

``` SQL
ORDER BY revenue DESC
LIMIT 10;
```

Implementing COALESCE and handling NULL values:

``` SQL
SELECT 
    COALESCE(city, country, 'other') AS city,
    COALESCE(country, 'other') AS country,
    SUM(COALESCE(total_transaction_revenue, 0)) AS total_revenue
FROM 
    all_sessions
WHERE 
    visit_type = 'PAGE'
    AND city IS NOT NULL
    AND country IS NOT NULL
    AND total_transaction_revenue IS NOT NULL
    AND total_transaction_revenue > 0
GROUP BY 
    city, country
ORDER BY 
    total_revenue DESC;
``` 

More complex query is unnecessary complicating everything, let's remain concise and effective. Also, price seems too high (incorrect input), let's divide it into 1M: 
``` SQL
SELECT  
    SUM(total_transaction_revenue/1000000) AS revenue, 
    city,
    country
FROM all_sessions
WHERE   
    visit_type = 'PAGE' 
    AND total_transaction_revenue > 0 
GROUP BY city, country
ORDER BY revenue DESC;
```

</details>


### ✅ Answer 

The country and city with the highest level of transaction revenue on the site is the United States and San Francisco with the total_transaction_revenue at $1198.12 (disregarding cities that were "not available in demo dataset").

## 💡Question 2: What is the average number of products ordered from visitors in each city and country?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

- Checking if 'product_quantity' is reliable
- A random product in the sales_report it has total_ordered '0', stock_level '73'. In products table, it has stock_level 73, but ordered_quantity 65, which signifies p.quantity should be renamed as 'ordered for restock'
  
``` SQL
SELECT p.product_sku, p.ordered_for_restock, sr.total_ordered, p.stock_level
FROM products p
JOIN sales_report sr USING(product_sku);
```

`all_sessions` doesn't have amount of products ordered (but just to make sure, I compared als.product_quantity to sr.total_ordered)

``` SQL
SELECT  als.country AS country, 
		CASE 
			WHEN als.city = '(not set)' THEN als.country
			WHEN als.city = 'not available in demo dataset' THEN als.country
			ELSE als.city
		END AS city,
		ROUND(AVG(sr.total_ordered)) AS avg_ordered_quantity
FROM all_sessions als
JOIN sales_report sr USING(product_sku) 
GROUP BY country, city
ORDER BY avg_ordered_quantity DESC;
``` 

How many countries/cities there are? Would they translate to rows in output? NOTE: not all countries have orders.

``` SQL
SELECT COUNT(*), country FROM all_sessions GROUP BY country ORDER BY count ASC;
SELECT COUNT(*), city FROM all_sessions GROUP BY city;
``` 

Output shows 136 countries + 266 cities = 402 rows in total, unless there are errors.
The problem here is that when cities are changed into countries, it's not adding up in country table and just remains in city column, producing too many results.

Average number of products ordered in each city:

``` SQL
SELECT  ROUND(AVG(sr.total_ordered)) AS avg_ordered_quantity,
		CASE 
			WHEN als.city = '(not set)' THEN als.country
			WHEN als.city = 'not available in demo dataset' THEN als.country
			ELSE als.city
		END AS city
FROM all_sessions als
JOIN sales_report sr USING(product_sku)
GROUP BY als.city, als.country;
-- WHERE (optionally)
	country != '(not set)' AND city != 'not available in demo dataset'
	AND city != '(not set)'
``` 

Average number of products ordered in each country:

``` SQL
SELECT  ROUND(AVG(sr.total_ordered)) AS avg_ordered_quantity,
		CASE 
			WHEN als.country = '(not set)' THEN 'not set'
			WHEN als.country = 'not available in demo dataset' THEN 'not set'
			ELSE als.country
		END AS country
FROM all_sessions als
JOIN sales_report sr USING(product_sku)
GROUP BY als.country
ORDER BY als.country;
``` 

Final query (filtered by visits):

``` SQL
SELECT  COALESCE(als.city, '(not set)') AS city,
	    COALESCE(als.country, '(not set)') AS country,
	    AVG(sr.total_ordered) AS avg_products_ordered,
	    COUNT(DISTINCT als.visitid) AS num_visits,
	    SUM(sr.total_ordered) AS total_products_ordered
FROM    all_sessions AS als
JOIN    sales_report AS sr ON als.product_sku = sr.product_sku
WHERE   als.city IS NOT NULL
    	AND als.country IS NOT NULL
    	AND sr.total_ordered > 0
GROUP BY als.city, als.country
-- HAVING 
    -- COUNT(DISTINCT als.visitid) > 1  -- to filter out cities/countries with only one visit
ORDER BY avg_products_ordered DESC
LIMIT 20;
```
</details>

### ✅ Answer (filtered by visits)
The output returns 348 rows with a combination of city, country and the corresponding average ordered quantity. Highest quantity was ordered from customers based in Hong Kong, Riyadh in Saudi Arabia and Brno in Czechia (319 each). NOTE: only orderes by visitors are taken into account.

## 💡Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT  DISTINCT als.country AS country,
		als.product_category2 AS product_category,
    	COUNT(DISTINCT als.visitid) AS num_visits,
    	SUM(sr.total_ordered) AS total_products_ordered
FROM    all_sessions AS als
JOIN    sales_report AS sr ON als.product_sku = sr.product_sku
WHERE   als.country IS NOT NULL AND country IN ('Canada') -- 'United States', 'United Kingdom')
        AND sr.total_ordered > 0
GROUP BY product_category, country
ORDER BY total_products_ordered DESC
LIMIT 5;
``` 
Checking top 5 categories in 3 countries (US, UK, CA) reveals that:
- in the US, people most frequently shop for home appliances (smart thermostats, home office, drinkware) and prefer to shop by brand (YouTube, Google)
- in UK, we can observe same trend with YouTube brand, drinkware and thermostats being in top 3, and bags making a cut as well
- Canadians prefer to buy drinkware, electronics, bags and thermostats; most popular brand is YouTube

For further analysis, we would compare what visitors from Canadian cities chose to search for.

``` SQL
SELECT  DISTINCT als.city AS city,
		als.product_category2 AS product_category,
    	COUNT(DISTINCT als.visitid) AS num_visits,
    	SUM(sr.total_ordered) AS total_products_ordered
FROM    all_sessions AS als
JOIN    sales_report AS sr ON als.product_sku = sr.product_sku
WHERE   country = 'Canada' AND als.city  <> 'not available in demo dataset' 
        AND sr.total_ordered > 0
GROUP BY city, product_category
ORDER BY city, total_products_ordered DESC;
```
</details>
    
### ✅ Answer
- results with 'not available in demo dataset' were excluded (34 out of 94)
- significant visits and product totals were not possible to calculate due to missing city
- remaining results point to Toronto being the most active, with leading categories such as drinkware, apparel and electronics 

## 💡Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
WITH ranked_products AS 
	(
    SELECT  
        als.city, 
        als.country, 
        als.product_sku, 
        als.product_name2, 
        SUM(sr.total_ordered) AS total_sold,
        ROW_NUMBER() OVER (PARTITION BY als.city, als.country ORDER BY SUM(sr.total_ordered) DESC) AS rank
    FROM all_sessions als
    JOIN sales_report sr USING (product_sku)
    WHERE 
        country <> '(not set)' 
        AND city NOT IN ('not available in demo dataset', '(not set)') 
        AND als.product_category2 <> '(not set)' 
    GROUP BY als.city, als.country, als.product_sku, als.product_category2, als.product_name2
	)
SELECT 
    city, 
    country, 
    product_sku, 
    product_name2, 
    total_sold
FROM ranked_products
WHERE rank = 1
ORDER BY total_sold DESC, country, city;
```
</details>

### ✅ Answer 

The output returns 233 rows (all cities listed where order were placed). Products in each city were ranked and only top result included in product_name and corresponding total_sold. 

Pattern in the products sold per city and country: 
1. Nest Cam Indoor Security Camera product and Google 17oz Bottle sell really well in terms of quantity in the US (Mountain View, New York, Palo Alto) and other countries
2. Surprisingly, "Ballpoint LED Light Pen" has top sales in Pune (India), Seoul and Boston (456 total sold in all three).

Accidently found an error in results, Singapore in France (1 row), and New York in Canada (1 row).

``` SQL
SELECT city, country FROM all_sessions
WHERE city = 'Singapore';

SELECT city, country FROM all_sessions
WHERE city = 'New York' and country = 'Canada';
```

## 💡Question 5: Can we summarize the impact of revenue generated from each city/country?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT  city, 
		country,
	    ROUND(100 * total_transaction_revenue / SUM(total_transaction_revenue) OVER (PARTITION BY country), 2) AS rev_share
FROM 
	(
	SELECT  city, 
			country,
			SUM(total_transaction_revenue) AS total_transaction_revenue
	FROM all_sessions
	  WHERE 
		 country <> '(not set)' AND city <> 'not available in demo dataset'
		AND city <> '(not set)' AND country = 'Canada'
		AND  total_transaction_revenue IS NOT NULL
	  GROUP BY city, country
	  ORDER BY SUM(total_transaction_revenue) DESC
	  ) AS sq
ORDER BY rev_share DESC;
```
</details>

### ✅ Answer
Built upon the query in Q1, Sydney, Tel Aviv and Zurich are the most impactful cities based on transaction revenue generated globally.
