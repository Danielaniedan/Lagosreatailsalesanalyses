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

## Repository Structure

```
.
â”œâ”€â”€ README.md
â”œâ”€â”€ queries.sql           # all SQL, one block per business question
â”œâ”€â”€ generate_charts.py    # rebuilds every chart from the database
â””â”€â”€ images/               # charts used in this report
```

Figures above are from the standard Chinook dataset (412 invoices, $2,328.60 total revenue). If your Q1 results differ, your file is a different version, so use your own numbers.*

