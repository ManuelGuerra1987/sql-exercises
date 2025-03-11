## SQL Exercises

### Table of contents

- [Question 1](#question-1)



## Question 1

**Highest-Grossing Items**

Assume you're given a table containing data on Amazon customers and their spending on products in different category, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

product_spend Example Input:

| Category     | Product          | User ID | Spend  | Transaction Date       |
|-------------|-----------------|---------|--------|------------------------|
| Appliance   | Refrigerator     | 165     | 246.00 | 12/26/2021 12:00:00    |
| Appliance   | Refrigerator     | 123     | 299.99 | 03/02/2022 12:00:00    |
| Appliance   | Washing Machine  | 123     | 219.80 | 03/02/2022 12:00:00    |
| Electronics | Vacuum           | 178     | 152.00 | 04/05/2022 12:00:00    |
| Electronics | Wireless Headset | 156     | 249.90 | 07/08/2022 12:00:00    |
| Electronics | Vacuum           | 145     | 189.00 | 07/15/2022 12:00:00    |


Example Output:

| Category     | Product          | Total Spend |
|-------------|-----------------|-------------|
| Appliance   | Refrigerator     | 299.99      |
| Appliance   | Washing Machine  | 219.80      |
| Electronics | Vacuum           | 341.00      |
| Electronics | Wireless Headset | 249.90      |


**Query:**

```sql

WITH cte AS (

  SELECT * FROM product_spend
  WHERE EXTRACT(YEAR FROM transaction_date) = 2022
),

cte2 AS(

  SELECT
    category,
    product,
    SUM(spend) as total_spend
  
  FROM cte 
  GROUP BY category,product

),

cte3 AS(

  SELECT 
  *,
  RANK() OVER (PARTITION BY category ORDER BY total_spend DESC) AS rank
  
  FROM cte2

)

SELECT category,product,total_spend FROM cte3
WHERE rank = 1 OR rank = 2;

```

**step by step**

cte2 output:

```sql

WITH cte AS (

  SELECT * FROM product_spend
  WHERE EXTRACT(YEAR FROM transaction_date) = 2022
),

cte2 AS(

  SELECT
    category,
    product,
    SUM(spend) as total_spend
  
  FROM cte 
  GROUP BY category,product
)
```  

| Category     | Product                | Total Spend |
|-------------|------------------------|-------------|
| Appliance   | Refrigerator            | 299.99      |
| Electronics | 3.5mm Headphone Jack    | 7.99        |
| Appliance   | Washing Machine         | 439.80      |
| Electronics | Computer Mouse          | 45.00       |
| Electronics | Vacuum                  | 486.66      |
| Appliance   | Microwave               | 49.99       |
| Electronics | Wireless Headset        | 467.89      |


cte3 output:

```sql

cte3 AS(

  SELECT 
  *,
  RANK() OVER (PARTITION BY category ORDER BY total_spend DESC) AS rank
  
  FROM cte2

)
```

| Category     | Product                | Total Spend | Rank |
|-------------|------------------------|-------------|------|
| Appliance   | Washing Machine         | 439.80      | 1    |
| Appliance   | Refrigerator            | 299.99      | 2    |
| Appliance   | Microwave               | 49.99       | 3    |
| Electronics | Vacuum                  | 486.66      | 1    |
| Electronics | Wireless Headset        | 467.89      | 2    |
| Electronics | Computer Mouse          | 45.00       | 3    |
| Electronics | 3.5mm Headphone Jack    | 7.99        | 4    |


final query output:

```sql

SELECT category,product,total_spend FROM cte3
WHERE rank = 1 OR rank = 2;

```

| Category     | Product          | Total Spend |
|-------------|-----------------|-------------|
| Appliance   | Washing Machine  | 439.80      |
| Appliance   | Refrigerator     | 299.99      |
| Electronics | Vacuum           | 486.66      |
| Electronics | Wireless Headset | 467.89      |
