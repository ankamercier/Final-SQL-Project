![image credit: Icon Scout](https://cdn3d.iconscout.com/3d/premium/thumb/search-for-product-on-e-commerce-website-6209340-5102567.png)
# Transforming and Analyzing Data with SQL

## Project/Goals
Following e-commerce database was created based on a demo dataset with raw records ranging from 462 to over 4 millions rows.  
Project aims to achieve following objectives:

- Build a bew database using an ecommerce dataset from csv file
- Investigate and perform data cleaning and transformation: remove irrelevant, duplicated data, fix structural errors, perform type conversion, handle missing data
- Look at the dataset metadata, analyze and recommend (if any) relations between the tables and columns and constraints, build an ERD
- Understand business needs based on given datapoints and metrics
- Write and execute queries to answer business questions
- Perform the QA process for data cleaning

Given 5 separate datasets (`all_sessions`, `analytics`, `products`, `sales_by_sku`, `sales_report`), the goal is to create a new PostgreSQL database called `ecommerce` by setting up tables and importing data into the tables from each .csv file. Then, perform data cleaning and transformation in a new database prior to answering any emerging business problems.

## Process
### 1. Importing raw data and engineering a database
### 2. Cleaning and transforming the data:
- The tables are cleaned in the following order: `all_sessions`, `analytics`, `products`, `sales_by_sku`, `sales_report`
- Cleaning was infact performed using a reverse method, by exploring the questions first, mapping the data, and discovering fixable issues during the process in order to reshape data without changing the content
- The overall approach to clean the data:
  -- Remove Unwanted or Irrelavent Observations
  -- Ensure Accurate Datatypes
  -- Address Missing Values
  -- Fix Typos
### 3. Answering initial questions
Answer a given set of questions to provide queries and explanations
### 4. Developing queries based on data exploration
Self-construct a set of questions related to the data, provide queries and explanations
### 5. Creating the ERD schema using Postgresql
### 6. Describe the QA process with queries and explanations

## Results
Data is well-organized and better structured.
(fill in what you discovered this data could tell you and how you used the data to answer those questions)

## Challenges 
-- repeatedly casting and converting data types for compatibility 
-- adjusting dates and times with offsets and format localization
-- renaming schemas, tables, and columns for clarity
-- identifying outliers and ambiguous columns

## Future Goals
(what would you do if you had more time?)

Disclaimer: this assignment is part of a final SQL project at Lighthouse Labs
