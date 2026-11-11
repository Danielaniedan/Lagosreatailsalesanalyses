# Lagos Retail Sales Analysis: A Benchmark Study with the Chinook Database

I wanted to practise turning business questions into SQL, so I used the Chinook database, a sample digital music store, as a benchmark for a retailer thinking about entering a new market like Lagos. I looked at where the money comes from, who the best customers are, what sells, and how the sales team performs, using joins, aggregations, CTEs and window functions.

> **Scope note:** Chinook has no Lagos or Nigerian customers. It is sample data for a fictional store, so the "Lagos" angle is a benchmarking exercise, not a study of the real Lagos market.

## Questions I Set Out to Answer

1. How much total revenue did the store make, and across how many invoices?
2. How do sales change over time?
3. Which countries and cities bring in the most money?
4. Who are the top-spending customers?
5. Who buys most often, and does frequent mean high-spending?
6. Which genres earn the most?
7. Which artists earn the most?
8. What are the best-selling tracks?
9. Which tracks have never sold a single copy?
10. How do the sales reps compare on customer revenue?

## Dataset

[Chinook database](https://github.com/lerocha/chinook-database) by Luis Rocha. It models a fictional digital music store: customers, employees, invoices, invoice lines, and a catalog of tracks, albums, artists and genres. The music data comes from an iTunes library, and the customers and sales are generated.

## Tools

- SQL (SQLite) and DB Browser for SQLite
- Python (matplotlib) for the charts
- Git and GitHub

## Findings

### 1. The USA is the biggest market, mostly because it has the most customers

![Top 10 countries by revenue](images/revenue_by_country.png)

- The USA brought in **$523.06**, well ahead of Canada ($303.96) and France ($195.10). The top 3 countries made up about 44% of total revenue.
- The gap is mostly about **customer count, not spending habits**. The USA has 13 customers and Canada 8, while countries with just one or two customers, like Chile ($46.62 from one customer) and the Czech Republic ($90.24 from two), spend about as much per head.
- **Takeaway for a new market:** revenue follows how many customers you win, so customer acquisition matters more than trying to get each customer to spend more.

### 2. Revenue is flat year to year

![Revenue by year](images/revenue_by_year.png)

- Yearly revenue stays in a narrow band of roughly **$450 to $481**, with a small peak in 2010 and no real growth trend.
- **Takeaway:** the store is stable but not growing, so any new-market plan needs a growth lever such as new customers or new products.

### 3. Rock carries the catalog

![Top 8 genres by revenue](images/revenue_by_genre.png)

- Rock earned about **$826.65**, roughly a third of all revenue, and the top artists mostly come from rock and metal.
- A large number of tracks have never sold, so the catalog is wider than demand. A focused launch catalog of proven sellers would be cheaper than copying the whole range.

![Top 10 artists by revenue](images/top_artists.png)

### 4. The sales team is closely matched

![Revenue managed by each sales rep](images/revenue_by_rep.png)

- Jane Peacock manages the most revenue ($833.04), followed by Margaret Park ($775.40) and Steve Johnson ($720.16). The spread between first and last is about 16%, so no single rep dominates.

### 5. Customers buy about equally often, so order size drives spend differences

- Almost every customer has the same number of invoices (about 7), so the "frequent buyer" idea doesn't separate customers here. Differences in lifetime spend come from basket size.

## What This Means for a Lagos Launch

- Plan around **winning customers**, not raising spend per customer.
- Start with a **small, proven catalog** and expand based on what actually sells.
- Test local tastes early. Rock dominance here may not carry over to Lagos.
-- =====================================================================
-- Chinook Sales Analysis: SQL Queries (SQLite syntax)
-- Source: https://github.com/lerocha/chinook-database
-- Each query lists the business question and the techniques it uses.
-- =====================================================================


-- Q1. Overall sales baseline
-- Question: How much revenue did the store make, across how many invoices?
-- Techniques: AGGREGATE (SUM, COUNT, AVG, MIN, MAX)
SELECT
    COUNT(*)              AS total_invoices,
    ROUND(SUM(Total), 2)  AS total_revenue,
    ROUND(AVG(Total), 2)  AS avg_invoice_value,
    MIN(InvoiceDate)      AS first_sale,
    MAX(InvoiceDate)      AS last_sale
FROM Invoice;


-- Q2. Revenue by year
-- Question: How do sales change over time?
-- Techniques: AGGREGATE, GROUP BY, ORDER BY
SELECT
    strftime('%Y', InvoiceDate) AS sales_year,
    COUNT(*)                    AS invoices,
    ROUND(SUM(Total), 2)        AS revenue
FROM Invoice
GROUP BY sales_year
ORDER BY sales_year;


-- Q3. Top 10 countries by revenue
-- Question: Which markets bring in the most money?
-- Techniques: INNER JOIN, AGGREGATE, GROUP BY, ORDER BY + LIMIT (top N)
SELECT
    c.Country,
    COUNT(DISTINCT c.CustomerId) AS customers,
    ROUND(SUM(i.Total), 2)       AS revenue
FROM Customer c
INNER JOIN Invoice i
    ON i.CustomerId = c.CustomerId
GROUP BY c.Country
ORDER BY revenue DESC
LIMIT 10;


-- Q4. Top 10 customers by lifetime spend
-- Question: Who are the highest-value customers?
-- Techniques: INNER JOIN, AGGREGATE, GROUP BY, ORDER BY + LIMIT (top N)
SELECT
    c.CustomerId,
    c.FirstName || ' ' || c.LastName AS customer,
    c.Country,
    ROUND(SUM(i.Total), 2)           AS total_spent
FROM Customer c
INNER JOIN Invoice i
    ON i.CustomerId = c.CustomerId
GROUP BY c.CustomerId, customer, c.Country
ORDER BY total_spent DESC
LIMIT 10;


-- Q5. Repeat buyers
-- Question: Which customers buy most often, and do frequent buyers spend more?
-- Techniques: INNER JOIN, GROUP BY + HAVING, AGGREGATE
SELECT
    c.CustomerId,
    c.FirstName || ' ' || c.LastName AS customer,
    COUNT(i.InvoiceId)               AS purchases,
    ROUND(SUM(i.Total), 2)           AS total_spent,
    ROUND(AVG(i.Total), 2)           AS avg_order_value
FROM Customer c
INNER JOIN Invoice i
    ON i.CustomerId = c.CustomerId
GROUP BY c.CustomerId, customer
HAVING COUNT(i.InvoiceId) >= 7
ORDER BY purchases DESC, total_spent DESC;


-- Q6. Genre performance
-- Question: Which genres earn the most?
-- Techniques: INNER JOIN (3 tables), GROUP BY + HAVING, AGGREGATE
SELECT
    g.Name                                     AS genre,
    SUM(il.Quantity)                           AS units_sold,
    ROUND(SUM(il.UnitPrice * il.Quantity), 2)  AS revenue
FROM InvoiceLine il
INNER JOIN Track t ON t.TrackId = il.TrackId
INNER JOIN Genre g ON g.GenreId = t.GenreId
GROUP BY g.GenreId, g.Name
HAVING SUM(il.Quantity) > 50
ORDER BY revenue DESC;


-- Q7. Top 10 artists by revenue
-- Question: Which artists contribute the most sales?
-- Techniques: INNER JOIN (4 tables), AGGREGATE, ORDER BY + LIMIT (top N)
SELECT
    ar.Name                                    AS artist,
    SUM(il.Quantity)                           AS units_sold,
    ROUND(SUM(il.UnitPrice * il.Quantity), 2)  AS revenue
FROM InvoiceLine il
INNER JOIN Track  t  ON t.TrackId  = il.TrackId
INNER JOIN Album  al ON al.AlbumId = t.AlbumId
INNER JOIN Artist ar ON ar.ArtistId = al.ArtistId
GROUP BY ar.ArtistId, ar.Name
ORDER BY revenue DESC
LIMIT 10;


-- Q8. Tracks that have never sold
-- Question: Which catalog items have zero purchases?
-- Techniques: LEFT JOIN (anti-join with IS NULL), ORDER BY
SELECT
    t.TrackId,
    t.Name AS track,
    g.Name AS genre
FROM Track t
LEFT JOIN InvoiceLine il ON il.TrackId = t.TrackId
LEFT JOIN Genre g        ON g.GenreId  = t.GenreId
WHERE il.InvoiceLineId IS NULL
ORDER BY genre, track;


-- Q9. Sales rep performance
-- Question: How do support reps compare on customers and revenue?
-- Techniques: LEFT JOIN (keeps reps with no customers), AGGREGATE, GROUP BY
SELECT
    e.EmployeeId,
    e.FirstName || ' ' || e.LastName AS employee,
    e.Title,
    COUNT(DISTINCT c.CustomerId)     AS customers,
    ROUND(COALESCE(SUM(i.Total), 0), 2) AS revenue_managed
FROM Employee e
LEFT JOIN Customer c ON c.SupportRepId = e.EmployeeId
LEFT JOIN Invoice  i ON i.CustomerId   = c.CustomerId
GROUP BY e.EmployeeId, employee, e.Title
ORDER BY revenue_managed DESC;


-- Q10. Where do our customers live?
-- Question: Which countries and cities do we actually serve?
-- Techniques: DISTINCT, ORDER BY
SELECT DISTINCT
    Country,
    City
FROM Customer
ORDER BY Country, City;


-- Q11. Reporting structure
-- Question: Who reports to whom in the team?
-- Techniques: SELF JOIN (join of my choice, INNER)
SELECT
    e.FirstName || ' ' || e.LastName AS employee,
    e.Title                          AS employee_title,
    m.FirstName || ' ' || m.LastName AS manager,
    m.Title                          AS manager_title
FROM Employee e
INNER JOIN Employee m
    ON m.EmployeeId = e.ReportsTo
ORDER BY manager, employee;


-- Q12. Markets with multiple customers, ranked by average spend
-- Question: Which countries with at least 2 customers spend the most per customer?
-- Techniques: CTE, INNER JOIN, GROUP BY + HAVING, AGGREGATE (AVG, COUNT)
WITH customer_spend AS (
    SELECT
        c.CustomerId,
        c.Country,
        SUM(i.Total) AS spend
    FROM Customer c
    INNER JOIN Invoice i
        ON i.CustomerId = c.CustomerId
    GROUP BY c.CustomerId, c.Country
)
SELECT
    Country,
    COUNT(*)              AS customers,
    ROUND(AVG(spend), 2)  AS avg_spend_per_customer
FROM customer_spend
GROUP BY Country
HAVING COUNT(*) >= 2
ORDER BY avg_spend_per_customer DESC;




