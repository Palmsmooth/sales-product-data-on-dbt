# Workflow for Working With SQL Queries
## step 1: Raw data

- raw data in bigquery

<img src="image\raw data.png" width=100% height=40%>

---

## step 2: Sales table

- create new files in models

<img src="image\click create file.png" width=100% height=40%>

<img src="image\create sales_table sql.png" width=100% height=40%>

- add code in sales_table.sql

```sh
SELECT *, (Quantity_Ordered * Price_Each) AS total_price
FROM `road-to-data-engineer-sql-db.dbtdemo.sales`
```

<img src="image\add code in sales_table sql.png" width=100% height=40%>

- we can click Preview to see what the data will look like if we run this code

<img src="image\click Preview.png" width=100% height=40%>

- if we want to see what sql this code compiles into when using sql + jinja, click on Compile

<img src="image\click Compile.png" width=100% height=40%>

- click Save before clicking Build

<img src="image\Save before clicking Build.png" width=100% height=40%>

- click Build, which works similarly to dbt run; however, unlike dbt run, it will also execute any defined tests

<img src="image\click Build.png" width=100% height=40%>

- we can view the status in dbt build --select sales_table

<img src="image\view the status.png" width=100% height=40%>

- in bigquery, a view will be created based on what we build in dbt

<img src="image\view sales_table in bigquery.png" width=100% height=40%>

---

## step 3: Total revenue
- create new files in models

<img src="image\click create file to total_revenue.png" width=100% height=40%>

<img src="image\create total_revenue sql.png" width=100% height=40%>

- add code in total_revenue.sql

```sh
SELECT SUM(total_price) AS total_rev
FROM {{ ref('sales_table') }}
WHERE Refunded IS FALSE
```

<img src="image\add code in total_revenue sql.png" width=100% height=40%>

- click Save before clicking Preview

<img src="image\click Save and click Preview in total_revenue sql.png" width=100% height=40%>

- click Compile

<img src="image\click Compile in total_revenue sql.png" width=100% height=40%>

- click Build

<img src="image\click Build in total_revenue sql.png" width=100% height=40%>

- we can view the status in dbt build --select total_revenue

<img src="image\view the status in total_revenue.png" width=100% height=40%>

- in bigquery, a view will be created based on what we build in dbt

<img src="image\view total_revenue in bigquery.png" width=100% height=40%>

- we can click on the Lineage tab to trace the upstream tables that the total_revenue table is derived from

<img src="image\click Lineage in total_revenue sql.png.png" width=100% height=40%>

---

## step 4: Popular products
- create new files in models

<img src="image\click create file to popular_products.png" width=100% height=40%>

<img src="image\create popular_products sql.png" width=100% height=40%>

- add code in popular_products.sql

```sh
{% set groups = ["batteries", "laptop", "headphone"] %}

WITH products_with_group AS
(SELECT
    *,
    CASE
    {% for group in groups %}
    WHEN lower(Product) LIKE '%{{ group }}%' THEN '{{group}}'
    {% endfor %}
    ELSE 'other'
    END AS product_group
FROM {{ ref('sales_table') }})
SELECT product_group, SUM(Quantity_Ordered) AS sale_count
FROM products_with_group
GROUP BY 1
ORDER BY sale_count DESC
```

<img src="image\add code in popular_products sql.png" width=100% height=40%>

- click Save before clicking Preview

<img src="image\click Preview in popular_products sql.png" width=100% height=40%>

- click Compile

<img src="image\click Compile in popular_products sql.png.png" width=100% height=40%>

- click Build

<img src="image\click Build in popular_products sql.png" width=100% height=40%>

- we can view the status in dbt build --select popular_products

<img src="image\view the status in popular_products.png" width=100% height=40%>

- in bigquery, a view will be created based on what we build in dbt

<img src="image\view popular_products in bigquery.png" width=100% height=40%>

- click Lineage

<img src="image\click Lineage in popular_products sql.png" width=100% height=40%>

- if we run dbt run --select sales_table+, it will execute the sales_table model along with all of its downstream dependencies

```sh
dbt run --select sales_table+
```
<img src="image\dbt run --select sales_table+.png" width=100% height=40%>

<img src="image\view the status in dbt run --select sales_table+.png" width=100% height=40%>

---

## step 5: Perform data quality testing
- create new files in models

<img src="image\click create file to tests_are_here yml.png" width=100% height=40%>

<img src="image\create tests_are_here yml.png" width=100% height=40%>

- add code in tests_are_here.yml

```sh
version: 2

models:
    - name: sales_table
      description: "ตารางข้อมูลการขาย"
      columns:
          - name: Order_ID
            description: "Primary Key"
            tests:
                - unique
                - not_null

    - name: popular_products
      description: "สินค้าขายดี"
      columns:
          - name: product_group
            description: "Primary Key"
            tests:
                - unique
                - not_null
```

<img src="image\add code in tests_are_here yml.png" width=100% height=40%>

- use the dbt build command to run all resources

```sh
dbt build
```
<img src="image\dbt build command to run all resources.png" width=100% height=40%>

- we can view the status in dbt build

<img src="image\view the status in dbt build.png" width=100% height=40%>

- if a test fails, such as unique_sales_table_Order_ID, we need to investigate the root cause and understand why it failed

- in this case, the failure occurs because sales_table stores the same Order_ID multiple times when a customer purchases multiple items under a single order, as shown in the example

<img src="image\Sales_December.png" width=100% height=40%>

- in this case, the data is not actually incorrect

- it’s expected behavior that the same Order_ID appears multiple times when an order contains multiple items. Therefore, this test may not be appropriate or logically aligned with the data model

---

## step 6: Create documentation and data lineage

- use the dbt docs generate command to generate project documentation

```sh
dbt docs generate
```
<img src="image\dbt docs generate command.png" width=100% height=40%>

- we can view the status in dbt docs generate

<img src="image\view the status in dbt docs generate.png" width=100% height=40%>

- click View Docs

<img src="image\click View Docs.png" width=100% height=40%>

- this will generate a documentation website for the project

<img src="image\website for the project.png" width=100% height=40%>

- we can search for models

<img src="image\search for models.png" width=100% height=40%>

---
