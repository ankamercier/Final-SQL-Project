![image credit: Icon Scout](https://cdn3d.iconscout.com/3d/premium/thumb/search-for-product-on-e-commerce-website-6209340-5102567.png)
# Transforming and Analyzing Data with SQL

## Project/Goals
Following e-commerce database was created based on a demo dataset with raw records ranging from 462 to over 4 millions rows [^1].  
Project aims to achieve following objectives:

- [x] Build a new database using an ecommerce dataset from csv file
- [x] Investigate and perform data cleaning and transformation: remove irrelevant, duplicated data, fix structural errors, perform type conversion, handle missing data
- [x] Look at the dataset metadata, analyze and recommend (if any) relations between the tables and columns and constraints, build an ERD
- [x] Understand business needs based on given datapoints and metrics
- [x] Write and execute queries to answer business questions
- [x] Perform the QA process for data cleaning
- [ ] Prevent alien invasion :alien:
- [ ] Celebrate with a magnificent bliss of antipasti and a glass of chianti once tasks are complete :wine_glass:

Given 5 separate tables (`all_sessions`, `analytics`, `products`, `sales_by_sku`, `sales_report`), the goal is to create a new PostgreSQL database called `ecommerce` by setting up tables and importing data into the tables from each .csv file. Then, perform data cleaning and transformation in a new database prior to answering any emerging business problems.

## Process
### 1. Importing raw data and engineering a database
During data extraction, raw data was exported from a source directly to a staging area. Taking into consideration high volume of one of the tables, all data transformation was performed directly in a database. Minor visualization was done in Excel, but preference was given to SQL to explore inconsistencies and clean the data. In order to perform SQL operations, pgAdmin was used as a database management tool. Normally for routine data manipulations (once data is cleaned and transformed), VS Code can be utilized as it has a more friendly interface and useful plugins. To work with repository, both Github desktop and command line were used.

> [!NOTE]
> All steps taken to create a database in pgAdmin were described in [data cleaning](https://github.com/ankamercier/Final-SQL-Project/blob/main/data_cleaning.md) 

### 2. Cleaning and transforming the data:

- The tables are cleaned in the following order: `all_sessions`, `analytics`, `products`, `sales_by_sku`, `sales_report`
- Cleaning was in fact performed using a reverse method, by exploring the questions first, mapping the data, and discovering fixable issues during the process in order to reshape data without changing the content

The overall approach to clean the data:
- Remove unwanted or irrelevant observations
- Ensure accurate datatypes
- Address missing values
- Fix typos
  
 :pushpin: **JUMP TO:** [data cleaning](https://github.com/ankamercier/Final-SQL-Project/blob/main/data_cleaning.md)

### 3. Answering initial questions
We all wish we could have all the answers at our fingertips. "What's for dinner?" or "Does Hogwarts really exist?" are sadly not part of this project. 
However, with a few simple queries, we can in fact challenge a given set of questions such as:
- Which cities and countries have the highest level of transaction revenues on the site?
- What is the average number of products ordered from visitors in each city and country?
- Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?
- What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?
- Can we summarize the impact of revenue generated from each city/country?
There is much more to explore! This database provides info on revenue by product as well as on how each visitor to the site interacted with the products (when they visited, where they were based, how many pages they viewed, how long they stayed on the site, and the number of transactions they entered).

:pushpin: **JUMP TO:** [initial business questions](https://github.com/ankamercier/Final-SQL-Project/blob/main/initial_business_questions.md)

### 4. Developing queries based on data exploration
Based on previous question and data cleaning stage, it was easy to self-construct a set of questions related to the data, and provide queries and explanations. 
Some of the questions include:
- What are the top 3 months with highest earning potential? What's the most popular month among visitors?
- How revenue segments of the company look like? Show quartiles divided into cities with revenue in millions.
- What are the top five products with the lowest average sentimentscore per product?

:pushpin: **JUMP TO:** [further data exploration](https://github.com/ankamercier/Final-SQL-Project/blob/main/further_data_exploration.md)

### 5. Creating the ERD schema using Postgresql 

<details>
  
<summary>click to view ERD below</summary>

![Rick roll](https://c.tenor.com/4gPD1ccxrVgAAAAC/rick-ashley-dance.gif)

### Gotcha!

You can check ERD in a section on the left. 

_(if you are still seeing this, I must have forgotten to take down Rick after practicing importing gifs to Github)_

</details>

### 6. Describe the QA process with queries and explanations

There are no universal solutions to conduct QA on any data. Adopting general guidelines and following data breadcrumbs helped to identify potential risk areas in given database.
Good data quality generates trust in data. In order to measure data's quality, the following DQ dimentions were assessed throughout the process [^2]:
- Completeness - the degree to which all records have data populated when expected
- Validity - measures the degree to which the values in a data element are valid
- Uniqueness measures the degree to which the records in a dataset are not duplicated
- Timeliness - the degree to which a dataset is available when expected and depends on service level agreements being set up between technical and business resources
- Consistency - a data quality dimension that measures the degree to which data is the same across all instances of the data
- Accuracy - grammatical errors, geographical correlations (e.g. Singapore in France), date differences (e.g. current workers' birthday 200 years ago 🧛)

:pushpin: **JUMP TO:** [quality assurance](https://github.com/ankamercier/Final-SQL-Project/blob/main/quality_assurance.md)

## Results
In general, data is well-organized and better structured. Queries run smooth. What else can Data Analyst wish for?..
(this section will br filled wiht discoveries at later stage, `check further_data_exploration` for insights)

## Challenges 
In general, data cleaning was substantially longer than drawing actual insights. Importance of pre-cleaning the data before importing the dataset and engineering the database has to be given more thought. Other issues included:
- Repeatedly casting and converting data types for compatibility
- Adjusting dates and times with offsets and format localization
- Renaming schemas, tables, and columns for clarity
- Identifying outliers and ambiguous columns
- Deciding on good PKs and FKs
- Finding out that my cat is incredibly good passive listener when it comes to complaints about badly organized tables

## Future Goals
What would you do if you had more time? First of all, remove the gifs. Then add a detailed explanation on how to use command line to push commit back to repository. Eventually I'd fix any typos and errors in code, come up with more questions and further simplify ERD (probably considering splitting tables for better view).

 > _Are you still reading? You must really like the formatting. Or the jokes. Without further ado, let the magic begin!_ :mage_man:

[^1]: Disclaimer: this assignment is part of a final SQL project at Lighthouse Labs
[^2]: [Datacamp: Data Quality Dimentions Cheat Sheet](https://www.datacamp.com/cheat-sheet/data-quality-dimensions-cheat-s)
