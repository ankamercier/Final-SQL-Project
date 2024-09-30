## 💡 Question 1: What are the top 3 months with highest earning potential? What's the most popular month among visitors?

### SQL queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT  COUNT(revenue), 
		EXTRACT(MONTH FROM visit_date) AS visit_month 
FROM analytics 
GROUP BY visit_month 
ORDER BY COUNT DESC LIMIT 3;

SELECT  COUNT( -- add artificial PK
		EXTRACT(MONTH FROM visit_date) AS visit_month 
FROM analytics 
GROUP BY visit_month 
ORDER BY COUNT DESC LIMIT 3;
```
</details>

### ✅ Answer
First of all, we should extract a month from visit_date in analytics table. In order to group earnings by month, we should use the aggregate function for revenue and subsequently group the results by month. To find out the top 3 best performing months (revenue-wise), we should order the results and limit them to 3. This will give us 3 months with highest earnings. Most popular months among visitors can be calculated using the same query with one small change - this time we would simply count all distinct rows based on artificial primary key assigned to analytics table. Results show May, June and July as the top 3 months.

## 💡 Question 2: How revenue segments of the company look like? Show quartiles divided into cities with revenue in millions.

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT 
    als.country,
    als.city,
    ROUND((als.total_transaction_revenue / 1000000), 2) AS revenue_in_millions, -- Convert to millions
    NTILE(4) OVER (ORDER BY als.total_transaction_revenue::numeric) AS revenue_quartile
  FROM all_sessions als
  WHERE als.total_transaction_revenue IS NOT NULL
    AND als.country != '(not set)'
    AND als.city != 'not available in demo dataset'
    AND als.city != '(not set)';
``` 

</details>

### ✅ Answer
The query divides the cities into four revenue quartiles based on their total transaction revenue. The data shows 56 unique city-country combinations with recorded transaction revenue.Revenue is converted to millions for easier interpretation.

#### Quartile distribution:
**1st Quartile (lowest revenue)**: bottom 3 rows include cities like New York ($7.50M), Mountain View ($8.98M), and San Francisco ($12.19M)
**4th Quartile (highest revenue)**: top 3 rows include Los Angeles ($363.00M), Tel Aviv-Yafo ($602.00M), and Sunnyvale ($649.24M)

#### Observations:
- There's a significant revenue gap between the lower and upper quartiles
- U.S. cities dominate the list, appearing in both low and high quartiles
- Some non-U.S. cities (like Tel Aviv-Yafo) appear in the highest revenue quartile
- The highest revenue city (Sunnyvale) generates about 86 times more revenue than the lowest in the list (New York)

## 💡 Question 3: Return a ranked list of products with a "positive" sentiment based on sentiment score (above 0.5) and magnitude higher than 1.

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT 
    sr.product_sku,
    sr.product_name,
    sr.total_ordered,
    p.sentiment_score,
    p.sentiment_magnitude,
    RANK() OVER (ORDER BY sr.total_ordered DESC) AS sales_rank,
    CASE 
      WHEN p.sentiment_score > 0.5 THEN 'Positive'
      WHEN p.sentiment_score < -0.5 THEN 'Negative'
      ELSE 'Neutral'
    END AS sentiment_category
  FROM sales_report sr
  JOIN products p ON sr.product_sku = p.product_sku
  WHERE p.sentiment_score > 0.5 AND p.sentiment_magnitude > 1 AND total_ordered > 0
  ORDER BY sales_rank DESC, sentiment_category;
``` 

  </details>

### ✅ Answer 
The query successfully identified products with positive sentiment (score > 0.5) and high magnitude (> 1), ranking them by total orders.
#### Sentiment Scores:
All products in the list have sentiment scores ranging from 0.6 to 0.9, indicating positive customer sentiment.
The highest sentiment score is 0.9, shared by several products including the Men's Bayside Graphic Tee, Mood Ninja Window Decal, and Men's Watershed Full Zip Hoodie Grey.
#### Sentiment Magnitude:
Sentiment magnitudes range from 1.2 to 1.8, showing strong intensity of sentiment.
The highest magnitude is 1.9 for the "Women's V-Neck Tee Charcoal", suggesting very strong positive sentiment.
#### Product Types:
The list includes a diverse range of products: apparel (vests, tees, hoodies, jackets), accessories (cap, backpack, journal), and home goods (window decal, smoke alarm). Apparel items seem to dominate the top of the list, indicating a generally positive reception of clothing products.

#### Sales Performance of the results with highest rank in "positive" sentiment category:
Interestingly, all products shown have a total_ordered value of highest a sales_rank of 64.
This suggests that while these products have positive sentiment, they may not have been ordered during the period covered by the sales data, or there might be an issue with the sales data recording.

> ### ❗ Here's what it means for business (insights and recommendations)
> The high sentiment scores and magnitudes indicate that customers who interact with these products have very positive impressions.
> The disconnect between positive sentiment and zero orders is noteworthy and requires further investigation. Possible explanations could include:
> - These are new or upcoming products with no sales data yet
> - There might be inventory or availability issues preventing sales
> - There could be a data recording problem in the sales system
> It would be valuable to cross-reference this data with inventory levels, product launch dates, and marketing efforts to understand why positively-received
> products aren't translating into sales. Suggestion for our fictional business would be to consider featuring these products more prominently in marketing
> campaigns, as they already have positive sentiment. Further actions business managers could take is to investigate any barriers to purchase for these items,
> such as pricing, availability, or visibility on the website.

## 💡 Question 4: Which top 3 marketing channels are the most effective in attracting unique visitors?

<details>

<summary> Expand to view code 👉 </summary>

SQL Queries:

``` SQL
SELECT  channel_grouping,
		COUNT(DISTINCT full_visitorid) AS unique_visitors
FROM all_sessions
GROUP BY channel_grouping
ORDER BY unique_visitors DESC
LIMIT 3;
``` 

</details>

### ✅ Answer 

Organic search leads bring in the total amount of unique visitors to the site with 8207 persons, followed by Direct and Referral (2805 and 2419 unique visitors).

## 💡 Question 5: What are the top five products with the lowest average sentimentscore per product?

### SQL Queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
SELECT  p.product_name, 
		AVG(s.sentiment_score) AS avg_sentiment_score
FROM sales_report s
JOIN products p USING (product_sku)
GROUP BY p.product_name
ORDER BY AVG(s.sentiment_score) ASC
LIMIT 5;
``` 

</details>

### ✅ Answer
To explore sentiment score towards products, we should join 2 tables: products and sales_report. Products with the lowest average sentimentscore are: Men's Vintage Henley, Android Women's Short Sleeve Badge Tee Dark Heather, Clip-on Compact Charger, Android Men's Long & Lean Badge Tee Charcoal, and Men's Vintage Badge Tee Sage. All listed products have an average sentiment_score below -0.2.

## 💡 Question 6: Combining results of Q2 and Q3 into a complex query to calculate market potential

### SQL queries

<details>

<summary> Expand to view code 👉 </summary>

``` SQL
WITH revenue_segments AS (
  SELECT 
    als.country,
    als.city,
    ROUND((als.total_transaction_revenue / 1000000), 2) AS revenue_in_millions, 
    NTILE(4) OVER (ORDER BY als.total_transaction_revenue::numeric) AS revenue_quartile
  FROM all_sessions als
  WHERE als.total_transaction_revenue IS NOT NULL
    AND als.country != '(not set)'
    AND als.city != 'not available in demo dataset'
    AND als.city != '(not set)'
),
product_performance AS (
  SELECT 
    sr.product_sku,
    sr.product_name,
    sr.total_ordered,
    p.sentiment_score,
    p.sentiment_magnitude,
    RANK() OVER (ORDER BY sr.total_ordered DESC) AS sales_rank,
    CASE 
      WHEN p.sentiment_score > 0.5 THEN 'Positive'
      WHEN p.sentiment_score < -0.5 THEN 'Negative'
      ELSE 'Neutral'
    END AS sentiment_category
  FROM sales_report sr
  JOIN products p ON sr.product_sku = p.product_sku
  WHERE total_ordered > 0
)
SELECT 
  rs.country, -- city
  rs.revenue_in_millions,
  rs.revenue_quartile,
  pp.product_sku,
  pp.product_name,
  pp.total_ordered,
  pp.sales_rank,
  pp.sentiment_category,
  -- AVG(a.time_on_site) OVER (PARTITION BY rs.country, rs.city) AS avg_time_on_site,
  COUNT(DISTINCT a.visitid) AS unique_visits,
  -- SUM(a.page_views) AS total_page_views,
  CASE 
    WHEN pp.total_ordered > 100 AND rs.revenue_quartile >= 3 THEN 'High Potential'
    WHEN pp.total_ordered > 50 AND rs.revenue_quartile >= 2 THEN 'Medium Potential'
    ELSE 'Low Potential'
  END AS market_potential
FROM revenue_segments rs
JOIN all_sessions als USING(country, city)
JOIN analytics a USING(visitid)
JOIN product_performance pp USING(product_sku)
WHERE pp.total_ordered > 100 AND rs.revenue_quartile >= 3
GROUP BY 
  rs.country, rs.city, rs.revenue_in_millions, rs.revenue_quartile,
  pp.product_sku, pp.product_name, pp.total_ordered, pp.sales_rank, pp.sentiment_category --, a.time_on_site
HAVING COUNT(DISTINCT a.visitid) > 1 -- AND market_potential LIKE 'High%'
ORDER BY rs.revenue_in_millions DESC
``` 

</details>

### ✅ Answer

This output combines revenue data, product performance, and customer engagement metrics to identify high-potential products and geographies, focusing on items with significant sales in high-revenue areas. Key Findings:
- all high-potential products are in the United States, suggesting a strong market focus or data bias towards the US market
- revenue ranges from $120 million to $649.24 million across different locations within the US; most entries fall into the 3rd and 4th revenue quartiles, as per the query's filtering criteria.
#### Top Products:
- Android 17oz Stainless Steel Sport Bottle (SKU: GGOEADHH073999)
- Cam Indoor Security Camera - USA (SKU: GGOENEBB078899)
- Cam Outdoor Security Camera - USA (SKU: GGOENEBQ078999)
- Sunglasses (SKU: GGOEGAAX0037)

#### Sales performance:
   - Total ordered quantities range from 102 to 167 units
   - The Android water bottle is the best-selling item (167 orders, rank 8)
   - Security cameras (both indoor and outdoor) are also popular

#### Sentiment analysis:
   - Most products have a "Neutral" sentiment category
   - The Outdoor Security Camera consistently shows a "Positive" sentiment

#### Customer engagement:
   - Unique visits vary from 2 to 10 across different product-location combinations
   - The Indoor Security Camera has the highest number of unique visits (10) in some locations

All listed items are categorized as "High Potential" due to the query's filtering criteria (total_ordered > 100 AND revenue_quartile >= 3)

> ### ❗Here's what it means for business (insights and recommendations):
> - Both indoor and outdoor security cameras show high potential. Business managers should consider expanding this product line or increasing marketing efforts.
> - The Outdoor Security Camera consistently receives positive sentiment. Investigate the factors contributing to this and apply insights to other products.
> - Same applies to the top-selling item: further questions should involve its marketing strategy, pricing, and features to replicate success with other products.
> - All high-potential products are in the US. What should be done to replicate this success in other countries, other markets?
> - Customer engagement is lagging and should be improved; most products have only 2 unique visits. It is recommended to develop strategies to increase site visits and customer interaction, especially for high-potential products.
> - Another avenue worth exploring is revenue variability: what are the factors contributing to the wide range of revenue across different locations for the same products?

