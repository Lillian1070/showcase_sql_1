# [SQL] 7-day Moving Average of Customer Payments

This SQL practice problem is based on **[LeetCode SQL 50 - 1321. Restaurant Growth](https://leetcode.com/problems/restaurant-growth/description/?envType=study-plan-v2&envId=top-sql-50)** 
- **Objective:** Calculate the 7-day moving average of customer payments to assess the feasibility of expansion.  
- **Practice Purpose:** Self-learning and reinforcement of SQL joins, aggregation, and window functions.
- **Outline:**
  - [**Practice**](#section-1) (practice problem and query output)
  - [**Solution**](#section-2) (step-by-step explanation)
  - [**Query Optimization**](#section-3) (refinement for efficiency and readability)


## <a name="section-1"></a>🧪 Practice

Table: `Customer`

| Column Name   | Type    |
|---------------|---------|
| customer_id   | int     |
| name          | varchar |
| visited_on    | date    |
| amount        | int     |

* In SQL,(`customer_id`, `visited_on`) is the primary key for this table.
* This table contains data about customer transactions in a restaurant.
* `visited_on` is the date on which the customer with ID (`customer_id`) has visited the restaurant.
* `amount` is the total paid by a customer.
 

You are the restaurant owner and you want to analyze a possible expansion (there will be at least one customer every day).

Compute the moving average of how much the customer paid in a seven days window (i.e., current day + 6 days before). `average_amount` should be rounded to two decimal places.

Return the result table ordered by `visited_on` in ascending order. The result format is in the following example.

 

### Example

#### Input: 

`Customer` table:

| customer_id | name         | visited_on   | amount      |
|-------------|--------------|--------------|-------------|
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 


#### Output: 

| visited_on   | amount       | average_amount |
|--------------|--------------|----------------|
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |


##### Explanation: 
* 1st moving average from 2019-01-01 to 2019-01-07 has an average_amount of (100 + 110 + 120 + 130 + 110 + 140 + 150)/7 = 122.86
* 2nd moving average from 2019-01-02 to 2019-01-08 has an average_amount of (110 + 120 + 130 + 110 + 140 + 150 + 80)/7 = 120
* 3rd moving average from 2019-01-03 to 2019-01-09 has an average_amount of (120 + 130 + 110 + 140 + 150 + 80 + 110)/7 = 120
* 4th moving average from 2019-01-04 to 2019-01-10 has an average_amount of (130 + 110 + 140 + 150 + 80 + 110 + 130 + 150)/7 = 142.86



## <a name="section-2"></a>🧠 Solution

*This section outlines my thought process for solving the problem.*

### Step 1: Identify the Fields Required

To create the result table with the 7-day moving average of customer payments, I need to find out the following info: 

* The current dates of each 7-day window (see temp table `T`)
* The total payments for each 7-day window (see step 4a calculation)


### Step 2: Create a Temporary Table `T`

To find out the current dates effectively, I create a temporary table `T` that finds all **unique** `visited_on` dates from the `Customer` table where the date is at least 6 days after the earliest recorded `visited_on` date. The goal is to start calculating the moving average from the 7th day onward to ensure each `visited_on` has a **full** 7-day window for the calculation.

```sql
WITH 
    T AS (
        SELECT DISTINCT visited_on
        FROM Customer
        WHERE visited_on >= (SELECT DATE_ADD(MIN(visited_on), INTERVAL 6 DAY) FROM Customer)
    )
```

The temporary table `T` should be similar to what we have below. 

| visited_on |
| ---------- |
| 2019-01-07 |
| 2019-01-08 |
| 2019-01-09 |
| 2019-01-10 |


### Step 3: Main Query

Once I have the current dates, I can draft the main query with the required variables and necessary conditions. 

* Listing out `visited_on`, `amount`, `average_amount` per example result table. Variables `amount` and `average_amount` will need further calculations in this case. 
* Using `WHERE` clause ensures that only `visited_on` dates from the temp table `T` are included, so only dates with a full 7-day window are considered.
* Grouping the results by `visited_on` allows the calculations of the sums and averages for each date.

```sql
SELECT 
    visited_on,
    amount, -- need further calculation
    average_amount -- need further calculation
FROM Customer
WHERE visited_on IN (SELECT visited_on FROM T)
GROUP BY visited_on;
```


### Step 4a: Calculation of `amount` (Subquery)

This subquery calculates the sum of the amount for each `visited_on` date within a 7-day window, including the current date (as `visited_on`).

```sql
(
    SELECT SUM(amount) 
    FROM Customer
    WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
) AS amount
```


### Step 4b: Calculation of `average_amount` (Subquery)

This subquery calculates the 7-day moving average by dividing the sum of the amount over the last 7 days by 7 and rounding it to 2 decimal places.

```sql
(
    SELECT ROUND(SUM(amount)/7, 2) 
    FROM Customer
    WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
) AS average_amount
```



#### Final Syntax and Output using MySQL

##### * Syntax

```sql
WITH 
    T AS (
        SELECT DISTINCT visited_on
        FROM Customer
        WHERE visited_on >= (SELECT DATE_ADD(MIN(visited_on), INTERVAL 6 DAY) FROM Customer)
    )
    
SELECT 
    visited_on,
    (
        SELECT SUM(amount) 
        FROM Customer 
        WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
    ) AS amount,
    (
        SELECT ROUND(SUM(amount)/7, 2) 
        FROM Customer 
        WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
    ) AS average_amount
FROM Customer c
WHERE visited_on IN (SELECT visited_on FROM T)
GROUP BY visited_on;
```


##### * Output

| visited_on | amount | average_amount |
| ---------- | ------ | -------------- |
| 2019-01-07 | 860    | 122.86         |
| 2019-01-08 | 840    | 120            |
| 2019-01-09 | 840    | 120            |
| 2019-01-10 | 1000   | 142.86         |



## <a name="section-3"></a>🛠️ Query Optimization using MySQL

*Note: This section is updated on 02/16/2025.*

While reviewing my SQL query, I realized that the `WHERE` condition could be incorporated into the main query without using a `WITH` clause, simplifying the SQL syntax.

Please note that `c.visited_on` specifically refers to the `visited_on` column from the `Customer` table in the main query, while `sub.visited_on` specifically refers to the `visited_on` column from the `Customer` table in the subquery at the second-to-last line. 

```sql
SELECT 
    visited_on,
    (
        SELECT SUM(amount) 
        FROM Customer 
        WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
    ) AS amount,
    (
        SELECT ROUND(SUM(amount)/7, 2) 
        FROM Customer 
        WHERE visited_on BETWEEN DATE_SUB(c.visited_on, INTERVAL 6 DAY) AND c.visited_on
    ) AS average_amount
FROM Customer c
WHERE visited_on >= (SELECT DATE_ADD(MIN(sub.visited_on), INTERVAL 6 DAY) FROM Customer sub)
GROUP BY visited_on;
```
