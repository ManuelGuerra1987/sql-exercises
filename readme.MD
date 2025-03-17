## SQL Exercises

### Table of contents

Medium:

- [Highest Grossing Items](#highest-grossing-items)
- [Top 5 Artists](#top-5-artists)
- [Card Launch Success](#card-launch-success)
- [International Call Percentage](#international-call-percentage)
- [Top Three Salaries](#top-three-salaries)

Hard:

- [Active User Retention](#active-user-retention)
- [3 Topping Pizzas](#3-topping-pizzas)
- [Y on Y Growth Rate](#y-on-y-growth-rate)
- [Repeated Payments](#repeated-payments)
- [Server Utilization Time](#server-utilization-time)
- [Department vs Company Salary](#department-vs-company-salary)


## Highest Grossing Items

[Link](https://datalemur.com/questions/sql-highest-grossing)

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


### Query:

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

### step by step

**cte2 output:**

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


**cte3 output:**

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


**final query output:**

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



## Top 5 Artists


Assume there are three Spotify tables: artists, songs, and global_song_rank, which contain information about the artists, songs, and music charts, respectively.

Write a query to find the top 5 artists whose songs appear most frequently in the Top 10 of the global_song_rank table. Display the top 5 artist names in ascending order, along with their song appearance ranking.

If two or more artists have the same number of song appearances, they should be assigned the same ranking, and the rank numbers should be continuous (i.e. 1, 2, 2, 3, 4, 5)


artists Example Input:

| Artist ID | Artist Name | Label Owner            |
|-----------|------------|------------------------|
| 101       | Ed Sheeran | Warner Music Group    |
| 120       | Drake      | Warner Music Group    |
| 125       | Bad Bunny  | Rimas Entertainment   |


songs Example Input:

| Song ID | Artist ID | Name           |
|---------|----------|---------------|
| 55511   | 101      | Perfect       |
| 45202   | 101      | Shape of You  |
| 22222   | 120      | One Dance     |
| 19960   | 120      | Hotline Bling |


global_song_rank Example Input:

| Day | Song ID | Rank |
|-----|---------|------|
| 1   | 45202   | 5    |
| 3   | 45202   | 2    |
| 1   | 19960   | 3    |
| 9   | 19960   | 15   |


### Query:


```sql

WITH cte AS (

SELECT 
  artists.artist_name,
  songs.name,
  global_song_rank.rank
FROM global_song_rank
JOIN songs ON global_song_rank.song_id = songs.song_id
JOIN artists ON artists.artist_id = songs.artist_id


),

cte2 AS (

SELECT * FROM cte
WHERE rank <= 10


),

cte3 AS (

SELECT artist_name,count(rank) AS counter 
FROM cte2
GROUP BY artist_name

),

cte4 AS (

SELECT 
  *,
  DENSE_RANK() OVER (ORDER BY counter DESC) as artist_rank

  FROM cte3
)

SELECT artist_name,artist_rank 
FROM cte4
WHERE artist_rank <= 5

```


### step by step

**cte2 output:**

```sql

WITH cte AS (

SELECT 
  artists.artist_name,
  songs.name,
  global_song_rank.rank
FROM global_song_rank
JOIN songs ON global_song_rank.song_id = songs.song_id
JOIN artists ON artists.artist_id = songs.artist_id


),

cte2 AS (

SELECT * FROM cte
WHERE rank <= 10


)

```


| Artist Name | Song Name      | Rank |
|-------------|--------------|------|
| Ed Sheeran  | Shape of You | 2    |
| Ed Sheeran  | Shape of You | 2    |
| Ed Sheeran  | Shape of You | 6    |
| Ed Sheeran  | Perfect      | 2    |
| Drake       | Hotline Bling | 3    |
| Bad Bunny   | Mia          | 9    |
| Bad Bunny   | Mia          | 7    |
| Bad Bunny   | Mia          | 7    |


**cte3 output:**

```sql

cte3 AS (

SELECT artist_name,count(rank) AS counter 
FROM cte2
GROUP BY artist_name

)
```


| Artist Name   | Counter |
|---------------|---------|
| Chris Brown   | 1       |
| Lady Gaga     | 3       |
| Katy Perry    | 2       |
| Adele         | 5       |
| Taylor Swift  | 8       |
| Drake         | 7       |
| Bad Bunny     | 7       |
| Ariana Grande | 1       |
| Ed Sheeran    | 5       |


**cte4 output:**

```sql

cte4 AS (

SELECT 
  *,
  DENSE_RANK() OVER (ORDER BY counter DESC) as artist_rank

  FROM cte3
)
```


| Artist Name   | Counter | Artist Rank |
|---------------|---------|-------------|
| Taylor Swift  | 8       | 1           |
| Bad Bunny     | 7       | 2           |
| Drake         | 7       | 2           |
| Ed Sheeran    | 5       | 3           |
| Adele         | 5       | 3           |
| Lady Gaga     | 3       | 4           |
| Katy Perry    | 2       | 5           |
| Ariana Grande | 1       | 6           |
| Chris Brown   | 1       | 6           |


**final query output:**

```sql

SELECT artist_name,artist_rank 
FROM cte4
WHERE artist_rank <= 5

```

| Artist Name   | Artist Rank |
|---------------|-------------|
| Taylor Swift  | 1           |
| Bad Bunny     | 2           |
| Drake         | 2           |
| Ed Sheeran    | 3           |
| Adele         | 3           |
| Lady Gaga     | 4           |
| Katy Perry    | 5           |



## Card Launch Success

[Link](https://datalemur.com/questions/card-launch-success)

Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month.

Before you can answer this question, you want to first get some perspective on how well new credit card launches typically do in their first month.

Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the monthly_cards_issued table for a given card. Order the results starting from the biggest issued amount.

monthly_cards_issued Example Input:

| Issue Month | Issue Year | Card Name              | Issued Amount |
|-------------|------------|------------------------|---------------|
| 1           | 2021       | Chase Sapphire Reserve | 170000        |
| 2           | 2021       | Chase Sapphire Reserve | 175000        |
| 3           | 2021       | Chase Sapphire Reserve | 180000        |
| 3           | 2021       | Chase Freedom Flex     | 65000         |
| 4           | 2021       | Chase Freedom Flex     | 70000         |


### Query:

```sql

WITH cte AS (
  SELECT 
    card_name,
    issued_amount,
    issue_month,
    issue_year,
    MAKE_DATE(issue_year, issue_month, 1) as date
  FROM monthly_cards_issued
),

cte2 AS (
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY card_name ORDER BY date ASC) as row_num
  FROM cte
)

SELECT 
  card_name,
  issued_amount
FROM cte2
WHERE row_num = 1
ORDER BY issued_amount DESC;

```

### step by step

**cte output:**

```sql

WITH cte AS (

SELECT 

  card_name,
  issued_amount,
  issue_month,
  issue_year,
  MAKE_DATE(issue_year, issue_month, 1) as date

  
FROM monthly_cards_issued

)

```

| Card Name              | Issued Amount | Issue Month | Issue Year | Date                  |
|------------------------|---------------|-------------|------------|-----------------------|
| Chase Sapphire Reserve | 160000        | 12          | 2020       | 12/01/2020 00:00:00   |
| Chase Sapphire Reserve | 170000        | 1           | 2021       | 01/01/2021 00:00:00   |
| Chase Sapphire Reserve | 175000        | 2           | 2021       | 02/01/2021 00:00:00   |
| Chase Sapphire Reserve | 180000        | 3           | 2021       | 03/01/2021 00:00:00   |
| Chase Freedom Flex     | 55000         | 1           | 2021       | 01/01/2021 00:00:00   |
| Chase Freedom Flex     | 60000         | 2           | 2021       | 02/01/2021 00:00:00   |
| Chase Freedom Flex     | 65000         | 3           | 2021       | 03/01/2021 00:00:00   |
| Chase Freedom Flex     | 70000         | 4           | 2021       | 04/01/2021 00:00:00   |
| Chase Sapphire Reserve | 150000        | 11          | 2020       | 11/01/2020 00:00:00   |


**cte2 output:**

```sql

cte2 AS (
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY card_name ORDER BY date ASC) as row_num
  FROM cte
)

```


| Card Name              | Issued Amount | Issue Month | Issue Year | Date                  | Row Num |
|------------------------|---------------|-------------|------------|-----------------------|---------|
| Chase Freedom Flex     | 55000         | 1           | 2021       | 01/01/2021 00:00:00   | 1       |
| Chase Freedom Flex     | 60000         | 2           | 2021       | 02/01/2021 00:00:00   | 2       |
| Chase Freedom Flex     | 65000         | 3           | 2021       | 03/01/2021 00:00:00   | 3       |
| Chase Freedom Flex     | 70000         | 4           | 2021       | 04/01/2021 00:00:00   | 4       |
| Chase Sapphire Reserve | 150000        | 11          | 2020       | 11/01/2020 00:00:00   | 1       |
| Chase Sapphire Reserve | 160000        | 12          | 2020       | 12/01/2020 00:00:00   | 2       |
| Chase Sapphire Reserve | 170000        | 1           | 2021       | 01/01/2021 00:00:00   | 3       |
| Chase Sapphire Reserve | 175000        | 2           | 2021       | 02/01/2021 00:00:00   | 4       |





**final query output:**

```sql

SELECT 
  card_name,
  issued_amount
FROM cte2
WHERE row_num = 1
ORDER BY issued_amount DESC

```

| Card Name              | Issued Amount |
|------------------------|---------------|
| Chase Sapphire Reserve | 150000        |
| Chase Freedom Flex     | 55000         |


## International Call Percentage

[Link](https://datalemur.com/questions/international-call-percentage)


A phone call is considered an international call when the person calling is in a different country than the person receiving the call.

What percentage of phone calls are international? Round the result to 1 decimal.

phone_calls Example Input:

| Caller ID | Receiver ID | Call Time            |
|-----------|-------------|----------------------|
| 1         | 2           | 2022-07-04 10:13:49 |
| 1         | 5           | 2022-08-21 23:54:56 |
| 5         | 1           | 2022-05-13 17:24:06 |
| 5         | 6           | 2022-03-18 12:11:49 |


phone_info Example Input:

| Caller ID | Country ID | Network  | Phone Number       |
|-----------|------------|----------|--------------------|
| 1         | US         | Verizon  | +1-212-897-1964    |
| 2         | US         | Verizon  | +1-703-346-9529    |
| 3         | US         | Verizon  | +1-650-828-4774    |
| 4         | US         | Verizon  | +1-415-224-6663    |
| 5         | IN         | Vodafone | +91 7503-907302    |
| 6         | IN         | Vodafone | +91 2287-664895    |


### Query:

```sql

WITH cte AS (

SELECT 

info1.country_id as caller,
info2.country_id as receiver

FROM phone_calls 
JOIN phone_info info1 ON phone_calls.caller_id = info1.caller_id
JOIN phone_info info2 ON phone_calls.receiver_id = info2.caller_id

),

cte2 AS (

SELECT
  *,
  CASE WHEN cte.caller != cte.receiver THEN 1 ELSE 0 END AS inter
  
  FROM cte

)

SELECT ROUND(CAST(SUM(inter) * 100.0 / COUNT(*) AS NUMERIC), 1) AS international_calls_pct 
FROM cte2

```

### step by step

**cte output:**

```sql

WITH cte AS (

SELECT 

info1.country_id as caller,
info2.country_id as receiver

FROM phone_calls 
JOIN phone_info info1 ON phone_calls.caller_id = info1.caller_id
JOIN phone_info info2 ON phone_calls.receiver_id = info2.caller_id

)

```


| Caller | Receiver |
|--------|----------|
| UK     | UK       |
| IN     | UK       |
| US     | UK       |
| US     | UK       |
| DE     | DE       |
| DE     | DE       |


**cte2 output:**

```sql

cte2 AS (

SELECT
  *,
  CASE WHEN cte.caller != cte.receiver THEN 1 ELSE 0 END AS inter
  
  FROM cte

)

```


| Caller | Receiver | Inter |
|--------|----------|-------|
| UK     | UK       | 0     |
| IN     | UK       | 1     |
| US     | UK       | 1     |
| US     | UK       | 1     |
| DE     | DE       | 0     |
| DE     | DE       | 0     |
| DE     | DE       | 0     |
| DE     | DE       | 0     |
| IN     | DE       | 1     |
| US     | US       | 1     |
| UK     | US       | 1     |



**final query output:**

```sql

SELECT ROUND(CAST(SUM(inter) * 100.0 / COUNT(*) AS NUMERIC), 1) as international_calls_pct 
FROM cte2;

```


| International Calls Percentage |
|---------------------------------|
| 54.5                            |



## Top Three Salaries

[Link](https://datalemur.com/questions/sql-top-three-salaries)


As part of an ongoing analysis of salary distribution within the company, your manager has requested a report identifying high earners in each department. A 'high earner' within a department is defined as an employee with a salary ranking among the top three salaries within that department.

You're tasked with identifying these high earners across all departments. Write a query to display the employee's name along with their department name and salary. In case of duplicates, sort the results of department name in ascending order, then by salary in descending order. If multiple employees have the same salary, then order them alphabetically.

Note: Ensure to utilize the appropriate ranking window function to handle duplicate salaries effectively.



employee Example Input:

| employee_id | name             | salary | department_id | manager_id |
|------------|----------------|--------|--------------|------------|
| 1          | Emma Thompson   | 3800   | 1            | 6          |
| 2          | Daniel Rodriguez| 2230   | 1            | 7          |
| 3          | Olivia Smith    | 2000   | 1            | 8          |
| 4          | Noah Johnson    | 6800   | 2            | 9          |
| 5          | Sophia Martinez | 1750   | 1            | 11         |
| 6          | Liam Brown      | 13000  | 3            | NULL       |
| 7          | Ava Garcia      | 12500  | 3            | NULL       |
| 8          | William Davis   | 6800   | 2            | NULL       |
| 9          | Isabella Wilson | 11000  | 3            | NULL       |
| 10         | James Anderson  | 4000   | 1            | 11         |



### Query:

```sql

WITH cte AS(

  SELECT 
    department_name,
    name,
    salary
    
  FROM employee JOIN department 
  ON employee.department_id = department.department_id

),

cte2 AS(

  SELECT
  *,
  DENSE_RANK() OVER (PARTITION BY department_name ORDER BY salary DESC) AS rank
  
  FROM cte
)

SELECT department_name,name,salary FROM cte2
WHERE rank <= 3
ORDER BY department_name ASC;

```


### step by step

**cte output:**

```sql

WITH cte AS(

  SELECT 
    department_name,
    name,
    salary
    
  FROM employee JOIN department 
  ON employee.department_id = department.department_id

)

```

| department_name     | name                | salary |
|--------------------|--------------------|--------|
| Data Analytics    | Amelia Lee         | 4000   |
| Data Analytics    | James Anderson     | 4000   |
| Data Analytics    | Sophia Martinez    | 1750   |
| Data Analytics    | Olivia Smith       | 7000   |
| Data Analytics    | Daniel Rodriguez   | 2230   |
| Data Analytics    | Emma Thompson      | 3800   |
| Data Science      | Logan Moore        | 8000   |
| Data Science      | Charlotte Miller   | 7000   |



**cte2 output:**

```sql

cte2 AS(

  SELECT
  *,
  DENSE_RANK() OVER (PARTITION BY department_name ORDER BY salary DESC) AS rank
  
  FROM cte
)

```

| department_name     | name               | salary | rank |
|--------------------|-------------------|--------|------|
| Data Analytics    | Olivia Smith      | 7000   | 1    |
| Data Analytics    | Amelia Lee        | 4000   | 2    |
| Data Analytics    | James Anderson    | 4000   | 2    |
| Data Analytics    | Emma Thompson     | 3800   | 3    |
| Data Analytics    | Daniel Rodriguez  | 2230   | 4    |
| Data Analytics    | Sophia Martinez   | 1750   | 5    |
| Data Engineering  | Liam Brown        | 13000  | 1    |
| Data Engineering  | Ava Garcia        | 12500  | 2    |
| Data Engineering  | Isabella Wilson   | 11000  | 3    |
| Data Engineering  | Mia Taylor        | 10800  | 4    |
| Data Engineering  | Benjamin Hernandez| 9500   | 5    |
| Data Science     | Logan Moore       | 8000   | 1    |
| Data Science     | Charlotte Miller  | 7000   | 2    |
| Data Science     | Noah Johnson      | 6800   | 3    |
| Data Science     | William Davis     | 6800   | 3    |


**final query output:**

```sql

SELECT department_name,name,salary FROM cte2
WHERE rank <= 3
ORDER BY department_name ASC;

```


| department_name    | name              | salary |
|-------------------|------------------|--------|
| Data Analytics   | Olivia Smith     | 7000   |
| Data Analytics   | Amelia Lee       | 4000   |
| Data Analytics   | James Anderson   | 4000   |
| Data Analytics   | Emma Thompson    | 3800   |
| Data Engineering | Liam Brown       | 13000  |
| Data Engineering | Ava Garcia       | 12500  |
| Data Engineering | Isabella Wilson  | 11000  |
| Data Science     | Logan Moore      | 8000   |
| Data Science     | Charlotte Miller | 7000   |
| Data Science     | Noah Johnson     | 6800   |
| Data Science     | William Davis    | 6800   |




## Active User Retention

[Link](https://datalemur.com/questions/user-retention)


Assume you're given a table containing information on Facebook user actions. Write a query to obtain number of monthly active users (MAUs) in July 2022, including the month in numerical format "1, 2, 3".

An active user is defined as a user who has performed actions such as 'sign-in', 'like', or 'comment' in both the current month and the previous month.


user_actionsExample Input:

| User ID | Event ID | Event Type | Event Date            |
|---------|---------|------------|------------------------|
| 445     | 7765    | sign-in    | 05/31/2022 12:00:00   |
| 742     | 6458    | sign-in    | 06/03/2022 12:00:00   |
| 445     | 3634    | like       | 06/05/2022 12:00:00   |
| 742     | 1374    | comment    | 06/05/2022 12:00:00   |
| 648     | 3124    | like       | 06/18/2022 12:00:00   |



### Query:

```sql

WITH cte AS (

SELECT 

*,
EXTRACT (MONTH FROM event_date) AS month,
EXTRACT (YEAR FROM event_date) AS year

FROM user_actions
)

select 
current.month,
count(DISTINCT current.user_id) as monthly_active_users
FROM cte current JOIN cte prev 
ON current.user_id = prev.user_id
AND current.year = prev.year
AND current.month = prev.month +1
WHERE current.month = 7 AND current.year = 2022
GROUP BY current.month

```


### step by step

**cte output:**

```sql


WITH cte AS (

SELECT 

*,
EXTRACT (MONTH FROM event_date) AS month,
EXTRACT (YEAR FROM event_date) AS year

FROM user_actions
)
```


| User ID | Event ID | Event Type | Event Date            | Month | Year |
|---------|---------|------------|------------------------|-------|------|
| 445     | 7765    | sign-in    | 05/31/2022 12:00:00   | 5     | 2022 |
| 445     | 3634    | like       | 06/05/2022 12:00:00   | 6     | 2022 |
| 648     | 3124    | like       | 06/18/2022 12:00:00   | 6     | 2022 |
| 648     | 2725    | sign-in    | 06/22/2022 12:00:00   | 6     | 2022 |
| 648     | 8568    | comment    | 07/03/2022 12:00:00   | 7     | 2022 |
| 445     | 4363    | sign-in    | 07/05/2022 12:00:00   | 7     | 2022 |
| 445     | 2425    | like       | 07/06/2022 12:00:00   | 7     | 2022 |
| 445     | 2484    | like       | 07/22/2022 12:00:00   | 7     | 2022 |


**Self join output:**

```sql

SELECT
  current.user_id,
  current.event_type,
  current.month,
  current.year,
  prev.event_type,
  prev.month,
  prev.year
  
FROM cte current JOIN cte prev 
ON current.user_id = prev.user_id
AND current.year = prev.year
AND current.month = prev.month +1
```

| User ID | Event Type | Month | Year | Event Type | Month | Year |
|---------|------------|-------|------|------------|-------|------|
| 445     | like       | 6     | 2022 | sign-in    | 5     | 2022 |
| 648     | comment    | 7     | 2022 | sign-in    | 6     | 2022 |
| 648     | comment    | 7     | 2022 | like       | 6     | 2022 |
| 445     | sign-in    | 7     | 2022 | like       | 6     | 2022 |
| 445     | like       | 7     | 2022 | like       | 6     | 2022 |
| 445     | like       | 7     | 2022 | like       | 6     | 2022 |
| 648     | sign-in    | 7     | 2022 | sign-in    | 6     | 2022 |
| 648     | sign-in    | 7     | 2022 | like       | 6     | 2022 |
| 445     | comment    | 7     | 2022 | like       | 6     | 2022 |


The self-join is used to find users who were active in consecutive months.

**final query output:**

```sql

select 
current.month,
count(DISTINCT current.user_id) as monthly_active_users
FROM cte current JOIN cte prev 
ON current.user_id = prev.user_id
AND current.year = prev.year
AND current.month = prev.month +1
WHERE current.month = 7 AND current.year = 2022
GROUP BY current.month

```


| Month | Monthly Active Users |
|-------|----------------------|
| 7     | 2                    |



## 3 Topping Pizzas

[Link](https://datalemur.com/questions/pizzas-topping-cost)


You’re a consultant for a major pizza chain that will be running a promotion where all 3-topping pizzas will be sold for a fixed price, and are trying to understand the costs involved.

Given a list of pizza toppings, consider all the possible 3-topping pizzas, and print out the total cost of those 3 toppings. Sort the results with the highest total cost on the top followed by pizza toppings in ascending order.

Break ties by listing the ingredients in alphabetical order, starting from the first ingredient, followed by the second and third.

Do not display pizzas where a topping is repeated. For example, ‘Pepperoni,Pepperoni,Onion Pizza’.

Ingredients must be listed in alphabetical order. For example, 'Chicken,Onions,Sausage'. 'Onion,Sausage,Chicken' is not acceptable.

pizza_toppings Example Input:

| Topping Name  | Ingredient Cost |
|--------------|----------------|
| Pepperoni    | 0.50           |
| Sausage      | 0.70           |
| Chicken      | 0.55           |
| Extra Cheese | 0.40           |


### Query:

```sql

WITH cte AS (
  
  SELECT 
  
    pizza1.topping_name AS topping1,
    pizza1.ingredient_cost AS cost1,
    pizza2.topping_name AS topping2,
    pizza2.ingredient_cost AS cost2,
    pizza3.topping_name AS topping3,
    pizza3.ingredient_cost AS cost3
  
  FROM pizza_toppings pizza1
  CROSS JOIN pizza_toppings pizza2
  CROSS JOIN pizza_toppings pizza3
  WHERE pizza1.topping_name < pizza2.topping_name
  AND pizza2.topping_name < pizza3.topping_name

)
SELECT 

  CONCAT(topping1,',',topping2,',',topping3) AS pizza_concat, 
  cost1+cost2+cost3 AS total_cost

FROM cte
ORDER BY total_cost DESC, pizza_concat ASC

```

It uses CROSS JOIN to combine every row with every other row three times, generating all possible three-topping combinations.

The WHERE condition ensures that each topping appears only once per combination and enforces an order (topping1 < topping2 < topping3) to prevent duplicates


**final query output:**

| Pizza Combination                         | Total Cost |
|-------------------------------------------|------------|
| Chicken, Pepperoni, Sausage               | 1.75       |
| Chicken, Extra Cheese, Sausage            | 1.65       |
| Extra Cheese, Pepperoni, Sausage          | 1.60       |
| Chicken, Sausage, Spinach                 | 1.55       |
| Chicken, Mushrooms, Sausage               | 1.50       |
| Chicken, Pineapple, Sausage               | 1.50       |
| Pepperoni, Sausage, Spinach               | 1.50       |



## Y on Y Growth Rate

[Link](https://datalemur.com/questions/yoy-growth-rate)


Assume you're given a table containing information about Wayfair user transactions for different products. Write a query to calculate the year-on-year growth rate for the total spend of each product, grouping the results by product ID.

The output should include the year in ascending order, product ID, current year's spend, previous year's spend and year-on-year growth percentage, rounded to 2 decimal places.

user_transactions Example Input:

| transaction_id | product_id | spend   | transaction_date       |
|----------------|------------|---------|------------------------|
| 1341           | 123424     | 1500.60 | 12/31/2019 12:00:00    |
| 1423           | 123424     | 1000.20 | 12/31/2020 12:00:00    |
| 1623           | 123424     | 1246.44 | 12/31/2021 12:00:00    |
| 1322           | 123424     | 2145.32 | 12/31/2022 12:00:00    |



### Query:

```sql


WITH cte AS (

SELECT

  EXTRACT(YEAR FROM transaction_date) AS year,
  product_id,
  SUM(spend) AS curr_year_spend

FROM user_transactions
GROUP BY year,product_id
ORDER BY year
),

cte2 AS (

  SELECT
    *,
    LAG(curr_year_spend) OVER (PARTITION BY product_id ORDER BY year) AS prev_year_spend
    
  FROM cte    

),

cte3 AS (

  SELECT
    *,
    ROUND((curr_year_spend / prev_year_spend - 1)*100,2) AS yoy_rate
  
  FROM cte2

)

SELECT * FROM cte3

```

### step by step

**cte output:**

```sql

WITH cte AS (

SELECT

  EXTRACT(YEAR FROM transaction_date) AS year,
  product_id,
  SUM(spend) AS curr_year_spend

FROM user_transactions
GROUP BY year,product_id
ORDER BY year
)
```

| year | product_id | curr_year_spend |
|------|------------|-----------------|
| 2019 | 123424     | 1500.60         |
| 2019 | 234412     | 1800.00         |
| 2019 | 543623     | 6450.00         |
| 2020 | 123424     | 1000.20         |
| 2020 | 234412     | 1234.00         |
| 2020 | 543623     | 5348.12         |
| 2021 | 123424     | 1246.44         |
| 2021 | 234412     | 889.50          |


**cte2 output:**


```sql

cte2 AS (

  SELECT
    *,
    LAG(curr_year_spend) OVER (PARTITION BY product_id ORDER BY year) AS prev_year_spend
    
  FROM cte    

)

```

| year | product_id | curr_year_spend | prev_year_spend |
|------|------------|-----------------|-----------------|
| 2019 | 123424     | 1500.60         | NULL            |
| 2020 | 123424     | 1000.20         | 1500.60         |
| 2021 | 123424     | 1246.44         | 1000.20         |
| 2022 | 123424     | 2145.32         | 1246.44         |
| 2019 | 234412     | 1800.00         | NULL            |
| 2020 | 234412     | 1234.00         | 1800.00         |
| 2021 | 234412     | 889.50          | 1234.00         |
| 2022 | 234412     | 2900.00         | 889.50          |
| 2019 | 543623     | 6450.00         | NULL            |
| 2020 | 543623     | 5348.12         | 6450.00         |
| 2021 | 543623     | 2345.00         | 5348.12         |
| 2022 | 543623     | 5680.00         | 2345.00         |


**cte3 output:**

```sql

cte3 AS (

  SELECT
    *,
    ROUND((curr_year_spend / prev_year_spend - 1)*100,2) AS yoy_rate
  
  FROM cte2

)

SELECT * FROM cte3
```

| year | product_id | curr_year_spend | prev_year_spend | yoy_rate |
|------|------------|-----------------|-----------------|----------|
| 2019 | 123424     | 1500.60         | NULL            | NULL     |
| 2020 | 123424     | 1000.20         | 1500.60         | -33.35   |
| 2021 | 123424     | 1246.44         | 1000.20         | 24.62    |
| 2022 | 123424     | 2145.32         | 1246.44         | 72.12    |
| 2019 | 234412     | 1800.00         | NULL            | NULL     |
| 2020 | 234412     | 1234.00         | 1800.00         | -31.44   |
| 2021 | 234412     | 889.50          | 1234.00         | -27.92   |
| 2022 | 234412     | 2900.00         | 889.50          | 226.03   |
| 2019 | 543623     | 6450.00         | NULL            | NULL     |
| 2020 | 543623     | 5348.12         | 6450.00         | -17.08   |
| 2021 | 543623     | 2345.00         | 5348.12         | -56.15   |
| 2022 | 543623     | 5680.00         | 2345.00         | 142.22   |




## Repeated Payments

[Link](https://datalemur.com/questions/repeated-payments)


Sometimes, payment transactions are repeated by accident; it could be due to user error, API failure or a retry error that causes a credit card to be charged twice.

Using the transactions table, identify any payments made at the same merchant with the same credit card for the same amount within 10 minutes of each other. Count such repeated payments.

The first transaction of such payments should not be counted as a repeated payment. This means, if there are two transactions performed by a merchant with the same credit card and for the same amount within 10 minutes, there will only be 1 repeated payment.

transactions Example Input:

| transaction_id | merchant_id | credit_card_id | amount | transaction_timestamp  |
|---------------|------------|----------------|--------|------------------------|
| 1             | 101        | 1              | 100    | 09/25/2022 12:00:00    |
| 2             | 101        | 1              | 100    | 09/25/2022 12:08:00    |
| 3             | 101        | 1              | 100    | 09/25/2022 12:28:00    |
| 4             | 102        | 2              | 300    | 09/25/2022 12:00:00    |
| 6             | 102        | 2              | 400    | 09/25/2022 14:00:00    |

Within 10 minutes after Transaction 1, Transaction 2 is conducted at Merchant 1 using the same credit card for the same amount. This is the only instance of repeated payment in the given sample data.


### Query:

```sql

WITH cte AS (

  SELECT
    *,
    LAG(transaction_timestamp) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_timestamp,
    LAG(merchant_id) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_merch,
    LAG(credit_card_id) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_cred,
    LAG(amount) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_amount
    
  FROM transactions 

),
cte2 AS (

  SELECT
    *,
    EXTRACT(EPOCH FROM (transaction_timestamp - prev_timestamp)) / 60 AS diff_minutes
    
    FROM cte

),


cte3 AS (

  SELECT
  *,
  CASE 
  
    WHEN merchant_id = prev_merch AND credit_card_id = prev_cred AND amount = prev_amount 
    AND diff_minutes <= 10 THEN 1 ELSE 0 END AS counter
  
  FROM cte2

)

select SUM(counter) AS payment_count from cte3

```


### step by step

**cte output:**

```sql

WITH cte AS (

  SELECT
    *,
    LAG(transaction_timestamp) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_timestamp,
    LAG(merchant_id) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_merch,
    LAG(credit_card_id) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_cred,
    LAG(amount) OVER (PARTITION BY merchant_id, credit_card_id ORDER BY transaction_timestamp) AS prev_amount
    
  FROM transactions 

)

```

This Common Table Expression (CTE) retrieves all transactions and uses the LAG() window function to get details of the previous transaction for each (merchant_id, credit_card_id) pair, ordered by transaction_timestamp. This allows us to compare each transaction with its previous one


| tran_id | mer_id | card_id | timestamp | amount | prev_tt    | prev_mer | prev_card | prev_amount |
|---------------|------------|----------------|------------------------|--------|----------------------|------------|-----------|-------------|
| 1             | 101        | 1              | 09/25/2022 12:00:00    | 100    | NULL                 | NULL       | NULL      | NULL        |
| 2             | 101        | 1              | 09/25/2022 12:08:00    | 100    | 09/25/2022 12:00:00  | 101        | 1         | 100         |
| 3             | 101        | 1              | 09/25/2022 12:28:00    | 100    | 09/25/2022 12:08:00  | 101        | 1         | 100         |
| 5             | 101        | 1              | 09/25/2022 13:37:00    | 100    | 09/25/2022 12:28:00  | 101        | 1         | 100         |
| 4             | 101        | 2              | 09/25/2022 12:20:00    | 300    | NULL                 | NULL       | NULL      | NULL        |
| 6             | 102        | 2              | 09/25/2022 14:00:00    | 400    | NULL                 | NULL       | NULL      | NULL        |
| 7             | 102        | 3              | 09/26/2022 10:00:00    | 300    | NULL                 | NULL       | NULL      | NULL        |
| 8             | 102        | 3              | 09/26/2022 10:10:00    | 300    | 09/26/2022 10:00:00  | 102        | 3         | 300         |
| 9             | 102        | 3              | 09/26/2022 10:14:00    | 300    | 09/26/2022 10:10:00  | 102        | 3         | 300         |
| 10            | 103        | 4              | 09/27/2022 12:00:00    | 50     | NULL                 | NULL       | NULL      | NULL        |
| 11            | 103        | 4              | 09/27/2022 12:09:00    | 50     | 09/27/2022 12:00:00  | 103        | 4         | 50          |
| 12            | 103        | 4              | 09/27/2022 22:00:00    | 50     | 09/27/2022 12:09:00  | 103        | 4         | 50          |
| 13            | 105        | 6              | 09/27/2022 12:00:00    | 200    | NULL                 | NULL       | NULL      | NULL        |
| 14            | 105        | 6              | 09/27/2022 12:10:00    | 100    | 09/27/2022 12:00:00  | 105        | 6         | 200         |


**cte2 output:**

```sql

cte2 AS (

  SELECT
    *,
    EXTRACT(EPOCH FROM (transaction_timestamp - prev_timestamp)) / 60 AS diff_minutes
    
    FROM cte

)

```

This CTE calculates the time difference (in minutes) between each transaction and its previous one.

| t_id | m_id | c_id | tt | amount | prev_tt       | prev_m | prev_c | prev_amo | diff |
|---------------|-------------|----------------|------------------------|--------|----------------------|------------|-----------|-------------|--------------|
| 1             | 101         | 1              | 09/25/2022 12:00:00    | 100    | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 2             | 101         | 1              | 09/25/2022 12:08:00    | 100    | 09/25/2022 12:00:00  | 101        | 1         | 100         | 8.0          |
| 3             | 101         | 1              | 09/25/2022 12:28:00    | 100    | 09/25/2022 12:08:00  | 101        | 1         | 100         | 20.0         |
| 5             | 101         | 1              | 09/25/2022 13:37:00    | 100    | 09/25/2022 12:28:00  | 101        | 1         | 100         | 69.0         |
| 4             | 101         | 2              | 09/25/2022 12:20:00    | 300    | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 6             | 102         | 2              | 09/25/2022 14:00:00    | 400    | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 7             | 102         | 3              | 09/26/2022 10:00:00    | 300    | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 8             | 102         | 3              | 09/26/2022 10:10:00    | 300    | 09/26/2022 10:00:00  | 102        | 3         | 300         | 10.0         |
| 9             | 102         | 3              | 09/26/2022 10:14:00    | 300    | 09/26/2022 10:10:00  | 102        | 3         | 300         | 4.0          |
| 10            | 103         | 4              | 09/27/2022 12:00:00    | 50     | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 11            | 103         | 4              | 09/27/2022 12:09:00    | 50     | 09/27/2022 12:00:00  | 103        | 4         | 50          | 9.0          |
| 12            | 103         | 4              | 09/27/2022 22:00:00    | 50     | 09/27/2022 12:09:00  | 103        | 4         | 50          | 591.0        |
| 13            | 105         | 6              | 09/27/2022 12:00:00    | 200    | NULL                 | NULL       | NULL      | NULL        | NULL         |
| 14            | 105         | 6              | 09/27/2022 12:10:00    | 100    | 09/27/2022 12:00:00  | 105        | 6         | 200         | 10.0         |



**cte3 output:**

```sql

cte3 AS (

  SELECT
  *,
  CASE 
  
    WHEN merchant_id = prev_merch AND credit_card_id = prev_cred AND amount = prev_amount 
    AND diff_minutes <= 10 THEN 1 ELSE 0 END AS counter
  
  FROM cte2

)
```
This CTE determines if a transaction should be counted as potentially fraudulent.

This CTE checks if the current transaction has the same merchant_id, credit_card_id, and amount as the previous transaction and the time difference (diff_minutes) is 10 minutes or less


| t_id | m_id | c_id | tt   | amount | prev_tt     | prev_me | prev_c | prev_am | diff | counter |
|----------------|-------------|-----------------|--------------------------|--------|-----------------------|------------|-----------|-------------|--------------|---------|
| 1              | 101         | 1               | 09/25/2022 12:00:00      | 100    | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 2              | 101         | 1               | 09/25/2022 12:08:00      | 100    | 09/25/2022 12:00:00   | 101        | 1         | 100         | 8.0          | 1       |
| 3              | 101         | 1               | 09/25/2022 12:28:00      | 100    | 09/25/2022 12:08:00   | 101        | 1         | 100         | 20.0         | 0       |
| 5              | 101         | 1               | 09/25/2022 13:37:00      | 100    | 09/25/2022 12:28:00   | 101        | 1         | 100         | 69.0         | 0       |
| 4              | 101         | 2               | 09/25/2022 12:20:00      | 300    | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 6              | 102         | 2               | 09/25/2022 14:00:00      | 400    | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 7              | 102         | 3               | 09/26/2022 10:00:00      | 300    | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 8              | 102         | 3               | 09/26/2022 10:10:00      | 300    | 09/26/2022 10:00:00   | 102        | 3         | 300         | 10.0         | 1       |
| 9              | 102         | 3               | 09/26/2022 10:14:00      | 300    | 09/26/2022 10:10:00   | 102        | 3         | 300         | 4.0          | 1       |
| 10             | 103         | 4               | 09/27/2022 12:00:00      | 50     | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 11             | 103         | 4               | 09/27/2022 12:09:00      | 50     | 09/27/2022 12:00:00   | 103        | 4         | 50          | 9.0          | 1       |
| 12             | 103         | 4               | 09/27/2022 22:00:00      | 50     | 09/27/2022 12:09:00   | 103        | 4         | 50          | 591.0        | 0       |
| 13             | 105         | 6               | 09/27/2022 12:00:00      | 200    | NULL                  | NULL       | NULL      | NULL        | NULL         | 0       |
| 14             | 105         | 6               | 09/27/2022 12:10:00      | 100    | 09/27/2022 12:00:00   | 105        | 6         | 200         | 10.0         | 0       |



**final query output:**

```sql

select SUM(counter) AS payment_count from cte3

```

Finally, the query sums up the counter column to get the total number of suspicious transactions

| payment_count |
|---------------|
| 4             |



## Server Utilization Time

[Link](https://datalemur.com/questions/total-utilization-time)


Amazon Web Services (AWS) is powered by fleets of servers. Senior management has requested data-driven solutions to optimize server usage.

Write a query that calculates the total time that the fleet of servers was running. The output should be in units of full days.

Assumptions:

- Each server might start and stop several times.
- The total time in which the server fleet is running can be calculated as the sum of each server's uptime.

server_utilization Example Input:


| server_id | session_status | status_time          |
|-----------|---------------|----------------------|
| 1         | start         | 08/02/2022 10:00:00 |
| 1         | stop          | 08/04/2022 10:00:00 |
| 1         | stop          | 08/13/2022 19:00:00 |
| 1         | start         | 08/13/2022 10:00:00 |
| 3         | stop          | 08/19/2022 10:00:00 |
| 3         | start         | 08/18/2022 10:00:00 |
| 5         | stop          | 08/19/2022 10:00:00 |
| 4         | stop          | 08/19/2022 14:00:00 |
| 4         | start         | 08/16/2022 10:00:00 |
| 3         | stop          | 08/14/2022 10:00:00 |
| 3         | start         | 08/06/2022 10:00:00 |
| 2         | stop          | 08/24/2022 10:00:00 |
| 2         | start         | 08/17/2022 10:00:00 |
| 5         | start         | 08/14/2022 21:00:00 |




### Query:

```sql


WITH cte AS (

  SELECT
    *,
    LAG(session_status) OVER (PARTITION BY server_id ORDER BY status_time) AS prev_status,
    LAG(status_time) OVER (PARTITION BY server_id ORDER BY status_time) AS prev_time
    
    FROM server_utilization

),

cte2 AS (

  SELECT
    *,
    CASE 
      WHEN session_status = 'stop' AND prev_status = 'start'
      THEN EXTRACT(EPOCH FROM (status_time - prev_time)) / 86400 END AS delta

  FROM cte

)

SELECT FLOOR(SUM(delta)) AS total_uptime_days FROM cte2

```


### step by step

**cte output:**

```sql

WITH cte AS (

  SELECT
    *,
    LAG(session_status) OVER (PARTITION BY server_id ORDER BY status_time) AS prev_status,
    LAG(status_time) OVER (PARTITION BY server_id ORDER BY status_time) AS prev_time
    
    FROM server_utilization

)

```

This CTE adds the previous row’s session_status for the same server_id and the previous row’s status_time for the same server_id


| server_id | session_status | status_time          | prev_status | prev_time           |
|-----------|---------------|----------------------|-------------|----------------------|
| 1         | start         | 08/02/2022 10:00:00 | NULL        | NULL                 |
| 1         | stop          | 08/04/2022 10:00:00 | start       | 08/02/2022 10:00:00 |
| 1         | start         | 08/13/2022 10:00:00 | stop        | 08/04/2022 10:00:00 |
| 1         | stop          | 08/13/2022 19:00:00 | start       | 08/13/2022 10:00:00 |
| 2         | start         | 08/17/2022 10:00:00 | NULL        | NULL                 |
| 2         | stop          | 08/24/2022 10:00:00 | start       | 08/17/2022 10:00:00 |
| 3         | start         | 08/06/2022 10:00:00 | NULL        | NULL                 |
| 3         | stop          | 08/14/2022 10:00:00 | start       | 08/06/2022 10:00:00 |
| 3         | start         | 08/18/2022 10:00:00 | stop        | 08/14/2022 10:00:00 |
| 3         | stop          | 08/19/2022 10:00:00 | start       | 08/18/2022 10:00:00 |
| 4         | start         | 08/16/2022 10:00:00 | NULL        | NULL                 |
| 4         | stop          | 08/19/2022 14:00:00 | start       | 08/16/2022 10:00:00 |
| 5         | start         | 08/14/2022 21:00:00 | NULL        | NULL                 |
| 5         | stop          | 08/19/2022 10:00:00 | start       | 08/14/2022 21:00:00 |



**cte2 output:**


```sql

cte2 AS (

  SELECT
    *,
    CASE 
      WHEN session_status = 'stop' AND prev_status = 'start'
      THEN EXTRACT(EPOCH FROM (status_time - prev_time)) / 86400 END AS delta

  FROM cte

)

```

This CTE filters the rows where the session_status is "stop" and the prev_status was "start" (indicating a transition from running to stopped).Then it calculates the downtime in days.


| server_id | session_status | status_time          | prev_status | prev_time           | delta                  |
|-----------|---------------|----------------------|-------------|----------------------|------------------------|
| 1         | start         | 08/02/2022 10:00:00 | NULL        | NULL                 | NULL                   |
| 1         | stop          | 08/04/2022 10:00:00 | start       | 08/02/2022 10:00:00 | 2.0000000000000000     |
| 1         | start         | 08/13/2022 10:00:00 | stop        | 08/04/2022 10:00:00 | NULL                   |
| 1         | stop          | 08/13/2022 19:00:00 | start       | 08/13/2022 10:00:00 | 0.37500000000000000000 |
| 2         | start         | 08/17/2022 10:00:00 | NULL        | NULL                 | NULL                   |
| 2         | stop          | 08/24/2022 10:00:00 | start       | 08/17/2022 10:00:00 | 7.0000000000000000     |
| 3         | start         | 08/06/2022 10:00:00 | NULL        | NULL                 | NULL                   |
| 3         | stop          | 08/14/2022 10:00:00 | start       | 08/06/2022 10:00:00 | 8.0000000000000000     |
| 3         | start         | 08/18/2022 10:00:00 | stop        | 08/14/2022 10:00:00 | NULL                   |
| 3         | stop          | 08/19/2022 10:00:00 | start       | 08/18/2022 10:00:00 | 1.00000000000000000000 |
| 4         | start         | 08/16/2022 10:00:00 | NULL        | NULL                 | NULL                   |
| 4         | stop          | 08/19/2022 14:00:00 | start       | 08/16/2022 10:00:00 | 3.1666666666666667     |
| 5         | start         | 08/14/2022 21:00:00 | NULL        | NULL                 | NULL                   |
| 5         | stop          | 08/19/2022 10:00:00 | start       | 08/14/2022 21:00:00 | 4.5416666666666667     |



**final query output:**

```sql

SELECT FLOOR(SUM(delta)) AS total_uptime_days FROM cte2

```

| total_uptime_days |
| ------------------|
| 26                |



## Department vs Company Salary

[Link](https://datalemur.com/questions/sql-department-company-salary-comparison)

The company wants to understand how the average salary in each department compares to the company's overall average salary each month.

Write a query to compare the average salary of employees in each department to the company's average salary for March 2024. Return the comparison result as 'higher', 'lower', or 'same' for each department. Display the department ID, payment month (in MM-YYYY format), and the comparison result.

employee Example Input:

| employee_id | name              | salary | department_id | manager_id |
|------------|------------------|--------|--------------|------------|
| 1          | Emma Thompson    | 3800   | 1            | 6          |
| 2          | Daniel Rodriguez | 2230   | 1            | 7          |
| 3          | Olivia Smith     | 7000   | 1            | 8          |
| 5          | Sophia Martinez  | 1750   | 1            | 11         |


salary Example Input:

| salary_id | employee_id | amount | payment_date         |
|-----------|------------|--------|----------------------|
| 1         | 1          | 3800   | 01/31/2024 00:00:00 |
| 2         | 2          | 2230   | 01/31/2024 00:00:00 |
| 3         | 3          | 7000   | 01/31/2024 00:00:00 |
| 4         | 4          | 6800   | 01/31/2024 00:00:00 |
| 5         | 5          | 1750   | 01/31/2024 00:00:00 |


### Query:

```sql

WITH cte AS (

  SELECT
    
    salary.amount,
    employee.department_id,
    TO_CHAR(payment_date, 'MM-YYYY') AS payment_date
    
  FROM salary 
  JOIN employee ON salary.employee_id = employee.employee_id

),

cte2 AS (

  SELECT
    *,
    AVG(amount) OVER (PARTITION BY department_id) AS avg_dep,
    AVG(amount) OVER () AS avg_global
    
  FROM cte  
),

cte3 AS (

  SELECT
    *,
    CASE WHEN avg_dep > avg_global THEN 'higher'
         WHEN avg_dep < avg_global THEN 'lower'
         ELSE 'same' END AS comparison
         
  FROM cte2       

)

SELECT DISTINCT 

  department_id,
  payment_date,
  comparison
  
FROM cte3
WHERE payment_date = '03-2024'

```

### step by step

**cte output:**

```sql

WITH cte AS (

  SELECT
    
    salary.amount,
    employee.department_id,
    TO_CHAR(payment_date, 'MM-YYYY') AS payment_date
    
  FROM salary 
  JOIN employee ON salary.employee_id = employee.employee_id

)
```


| amount | department_id | payment_date |
|--------|--------------|--------------|
| 3800   | 1            | 03-2024      |
| 3800   | 1            | 02-2024      |
| 3800   | 1            | 01-2024      |
| 7000   | 1            | 01-2024      |
| 6800   | 2            | 03-2024      |
| 6800   | 2            | 02-2024      |
| 6800   | 2            | 01-2024      |


**cte2 output:**

```sql

cte2 AS (

  SELECT
    *,
    AVG(amount) OVER (PARTITION BY department_id) AS avg_dep,
    AVG(amount) OVER () AS avg_global
    
  FROM cte  
)

```

| amount | department_id | payment_date | avg_dep          | avg_global        |
|--------|--------------|--------------|------------------|------------------|
| 3800   | 1            | 03-2024      | 3796.6666666667  | 7212.0000000000  |
| 4000   | 1            | 01-2024      | 3796.6666666667  | 7212.0000000000  |
| 4000   | 1            | 02-2024      | 3796.6666666667  | 7212.0000000000  |
| 2230   | 1            | 02-2024      | 3796.6666666667  | 7212.0000000000  |
| 2230   | 1            | 01-2024      | 3796.6666666667  | 7212.0000000000  |
| 8000   | 2            | 03-2024      | 7150.0000000000  | 7212.0000000000  |
| 6800   | 2            | 03-2024      | 7150.0000000000  | 7212.0000000000  |
| 6800   | 2            | 02-2024      | 7150.0000000000  | 7212.0000000000  |
| 6800   | 2            | 01-2024      | 7150.0000000000  | 7212.0000000000  |
| 10800  | 3            | 03-2024      | 11360.0000000000 | 7212.0000000000  |
| 10800  | 3            | 02-2024      | 11360.0000000000 | 7212.0000000000  |
| 10800  | 3            | 01-2024      | 11360.0000000000 | 7212.0000000000  |
| 9500   | 3            | 03-2024      | 11360.0000000000 | 7212.0000000000  |
| 9500   | 3            | 02-2024      | 11360.0000000000 | 7212.0000000000  |
| 9500   | 3            | 01-2024      | 11360.0000000000 | 7212.0000000000  |



**cte3 output:**

```sql

cte3 AS (

  SELECT
    *,
    CASE WHEN avg_dep > avg_global THEN 'higher'
         WHEN avg_dep < avg_global THEN 'lower'
         ELSE 'same' END AS comparison
         
  FROM cte2       

)
```

| amount | department_id | payment_date | avg_dep | avg_global | comparison |
|--------|--------------|--------------|---------|------------|------------|
| 3800   | 1            | 03-2024      | 3796.67 | 7212.00    | lower      |
| 4000   | 1            | 01-2024      | 3796.67 | 7212.00    | lower      |
| 4000   | 1            | 02-2024      | 3796.67 | 7212.00    | lower      |
| 2230   | 1            | 02-2024      | 3796.67 | 7212.00    | lower      |
| 2230   | 1            | 01-2024      | 3796.67 | 7212.00    | lower      |
| 8000   | 2            | 03-2024      | 7150.00 | 7212.00    | lower      |
| 6800   | 2            | 03-2024      | 7150.00 | 7212.00    | lower      |
| 6800   | 2            | 02-2024      | 7150.00 | 7212.00    | lower      |
| 6800   | 2            | 01-2024      | 7150.00 | 7212.00    | lower      |
| 10800  | 3            | 03-2024      | 11360.00 | 7212.00    | higher     |
| 10800  | 3            | 02-2024      | 11360.00 | 7212.00    | higher     |
| 10800  | 3            | 01-2024      | 11360.00 | 7212.00    | higher     |
| 9500   | 3            | 03-2024      | 11360.00 | 7212.00    | higher     |
| 9500   | 3            | 02-2024      | 11360.00 | 7212.00    | higher     |
| 9500   | 3            | 01-2024      | 11360.00 | 7212.00    | higher     |


**final query output:**

```sql

SELECT DISTINCT 

  department_id,
  payment_date,
  comparison
  
FROM cte3
WHERE payment_date = '03-2024'

```


| department_id | payment_date | comparison |
|---------------|--------------|------------|
| 1             | 03-2024      | lower      |
| 2             | 03-2024      | lower      |
| 3             | 03-2024      | higher     |
