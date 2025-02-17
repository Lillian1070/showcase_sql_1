# lc_sql50_subq_4

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

optimization

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
WHERE visited_on >= (SELECT DATE_ADD(MIN(visited_on), INTERVAL 6 DAY) FROM Customer)
GROUP BY visited_on;
```
