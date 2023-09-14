
# E-Commerce Dimensional Modelling

In my Instacart data project, I've implemented a dimensional model using a star schema. This involved transforming data from various CSV files into fact and dimension tables. The star schema design consists of a central fact table surrounded by dimension tables, enabling efficient querying. The fact table, named fact_order_products, holds transactional data, while dimension tables like dim_products and dim_orders provide additional context. This structured approach facilitates easy retrieval of insights, making it a powerful tool for analysis in my Instacart data project.


## Prerequisites
Before you begin, ensure you have met the following requirements:
- You have a Windows/Linux/Mac machine.
- You have Amazon Web Services Account.
- You have Snowflake Account.

## Steps for Dimensional Modelling

1. Go to Kaggle and download the [dataset](https://www.kaggle.com/competitions/instacart-market-basket-analysis/data)

2. You can also download the dataset from [google drive](https://drive.google.com/drive/folders/1XJluibMqtv5Ulw3R7nSqQWXi5e6s5FUQ)

3. Login to you AWS Consle and create S3 bucket and upload the data into instacart folder in S3 bucket.![alt text](https://github.com/battaprikshit/E-Commerce-Dimensional-Modelling/blob/main/instacart%20Screenshorts/sc1.png)

4. Login to Snowflake and Create a Database . I have created database named Instacart.
5. Create Stage to Connect AWS S3 Bucket to Snowflake Database

```sql
CREATE STAGE my_stage
URL = 's3://'
CREDENTIALS = (AWS_KEY_ID='YOUR_AWS_KEY_ID'
               AWS_SECRET_KEY='YOUR_AWS_SECRET_KEY');
```
6. Create File Format Conditions. This will create the conditions when we want to load the data from S3 to Snowflake.
```sql 
CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE ='CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"';
```
7.  Next step is to create and load the data to all the tables
```sql
-  Create and load data into aisles table

CREATE TABLE aisles (
        aisle_id INTEGER PRIMARY KEY,
        aisle VARCHAR
    );

COPY INTO aisles (aisle_id, aisle) 
from@my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME= 'csv_file_format')

--  Create and Load Data to departments table

CREATE TABLE departments (department_id INTEGER PRIMARY KEY,
department VARCHAR);

SELECT * FROM DEPARTMENTS

COPY INTO departments (department_id, department) 
from@my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME= 'csv_file_format')

--  Create and load data to Orders table

CREATE OR REPLACE TABLE orders (
        order_id INTEGER PRIMARY KEY,
        user_id INTEGER,
        eval_set STRING,
        order_number INTEGER,
        order_dow INTEGER,
        order_hour_of_day INTEGER,
        days_since_prior_order INTEGER
    );

COPY INTO orders (order_id,user_id,eval_set,order_number,order_dow,order_hour_of_day,days_since_prior_order) 
from@my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME= 'csv_file_format')


--  Create and load data to products table

CREATE OR REPLACE TABLE products (
        product_id INTEGER PRIMARY KEY,
        product_name VARCHAR,
        aisle_id INTEGER,
        department_id INTEGER
    );

COPY INTO products (product_id, product_name, aisle_id, department_id)
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

--  Create and load data to order_products table

CREATE OR REPLACE TABLE order_products (
        order_id INTEGER,
        product_id INTEGER,
        add_to_cart_order INTEGER,
        reordered INTEGER,
        PRIMARY KEY (order_id, product_id)
    );
    
COPY INTO order_products (order_id, product_id, add_to_cart_order, reordered)
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

```
8. Next Step is we have to create the Fact and Dimension tables. First we can create the model for the tables. ---url

9. Now As per our dimensional model we can create the facts and dimension tables and load the data into these tables.

10. The script to create the fact and dimension table is as
```sql 

CREATE or REPLACE TABLE dim_user as (select user_id from orders);
select * from dim_user;

CREATE OR REPLACE TABLE dim_products as (select product_id,product_name from products);

CREATE OR REPLACE TABLE dim_department as (select department_id,department from departments);

CREATE OR REPLACE TABLE dim_aisles as (select aisle_id,aisle from aisles);

CREATE OR REPLACE TABLE dim_orders as (select order_id,order_number,order_dow,order_hour_of_day, days_since_prior_order from orders);

CREATE OR REPLACE TABLE fact_order_products as (select op.order_id,op.product_id,o.user_id,p.aisle_id,p.department_id,op.add_to_cart_order,op.reordered from order_products op join products p on op.product_id = p.product_id join orders o on op.order_id= o.order_id);

```

## Usage/ Analytics

1. We have create the fact and dimenion table and now we can use these tables to query or data.
```sql
-- Query to calculate the total number of products ordered per department:
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_department d ON fop.department_id = d.department_id
GROUP BY
  d.department;

![alt text](https://github.com/battaprikshit/E-Commerce-Dimensional-Modelling/blob/main/instacart%20Screenshorts/sc2.png)


-- Query to find the top 5 aisles with the highest number of reordered products:
SELECT
  a.aisle,
  COUNT(*) AS total_reordered
FROM
  fact_order_products fop
JOIN
  dim_aisles a ON fop.aisle_id = a.aisle_id
WHERE
  fop.reordered = TRUE
GROUP BY
  a.aisle
ORDER BY
  total_reordered DESC
LIMIT 5;
```

![alt text](https://github.com/battaprikshit/E-Commerce-Dimensional-Modelling/blob/main/instacart%20Screenshorts/sc3.png)

-- Query to calculate the average number of products added to the cart per order by day of the week:
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;

![alt text](https://github.com/battaprikshit/E-Commerce-Dimensional-Modelling/blob/main/instacart%20Screenshorts/sc4.png)

-- Query to identify the top 10 users with the highest number of unique products ordered:
SELECT
  u.user_id,
  COUNT(DISTINCT fop.product_id) AS unique_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_user u ON fop.user_id = u.user_id
GROUP BY
  u.user_id
ORDER BY
  unique_products_ordered DESC
LIMIT 10;
```

As we can see we can do Analytics on top of these tables.
 
 

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are greatly appreciated. If you have a suggestion that would make this better, please fork the repo and create a pull request. Don't forget to give the project a star! Thanks again!

1. Fork this repository.
2. Create a branch: `git checkout -b <branch_name>`.
3. Make your changes and commit them: `git commit -m '<commit_message>'`
4. Push to the original branch: `git push origin <project_name>/<location>`
5. Create the pull request.

Alternatively see the GitHub documentation on [creating a pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request). 


## License


Distributed under the MIT License. See LICENSE.txt for more information.
