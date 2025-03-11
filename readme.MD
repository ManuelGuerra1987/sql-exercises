## SQL Exercises

### Table of contents

Medium:

- [Question 1](#question-1)
- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)



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



## Question 2

**Top 5 Artists**

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


**Query:**


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

select artist_name,count(rank) as counter from cte2
group by artist_name

),

cte4 AS (

SELECT 
  *,
  DENSE_RANK() OVER (ORDER BY counter DESC) as artist_rank

  FROM cte3
)

select artist_name,artist_rank from cte4
WHERE artist_rank <= 5

```


**Step by step:**

cte2 output:

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


cte3 output:

```sql

cte3 AS (

select artist_name,count(rank) as counter from cte2
group by artist_name

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


cte4 output:

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


final query output:

```sql

select artist_name,artist_rank from cte4
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



## Question 3

**Card Launch Success**

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


**Query:**

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

**Step by step:**

cte output:

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


cte2 output:

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





final query output:

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


## Question 4

**International Call Percentage**

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


**Query:**

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

select ROUND(CAST(SUM(inter) * 100.0 / COUNT(*) AS NUMERIC), 1) as international_calls_pct from cte2

```

**Step by step:**

cte output:

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


cte2 output:

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



final query output:

```sql

SELECT ROUND(CAST(SUM(inter) * 100.0 / COUNT(*) AS NUMERIC), 1) as international_calls_pct 
FROM cte2;

```


| International Calls Percentage |
|---------------------------------|
| 54.5                            |
