# Sakila-DVD-Rental-Business-Analysis


## Table of Contents
1. [Project Overview](#project-overview)
2. [Data Sources](#data-sources)
3. [Tools](#tools)
4. [Data Preparation](#data-preparation)
5. [Exploratory Data Analysis](#exploratory-data-analysis)
6. [Operational Dashboard](#operational-dashboard)
7. [Data Analyis](#data-analysis)
8. [Business Insights](#business-insights)
9. [Recommendations](#recommendations)


## Project Overview 
---

This project aims to provide actionable insights into the operational performance and revenue drivers of a legacy DVD rental business using the Sakila database. By analyzing rental patterns, inventory health, and geographic data, this analysis identifies opportunities to reduce losses from late returns and optimize inventory for high-demand categories.


## Data Sources

The primary dataset used is the Sakila Sample Database, which represents a DVD rental store's business processes including films, inventory, customers, and payments.

## Tools

- **MySQL/SQL Server** - Data Extraction and Analysis

- **Power BI** - Creating Interactive Dashboards and Maps

- **Excel** - Supplementary Data Inspection


## Data Preparation

In the initial phase, the following tasks were performed:

- **Data Inspection:** Verified relationships between the rental, payment, and inventory tables.

- **Handling Missing Values:** Filtered out records where return_date was NULL to ensure accurate "Late Return" calculations.

- **Data Formatting:** Converted date strings into usable datetime formats for `DATEDIFF` calculations.


## Exploratory Data Analysis

EDA was conducted to answer key business questions:

1. Which film categories generate the highest revenue?

2. What is the percentage of late returns across different genres?

3. Which geographic regions hold the highest customer concentration?



## Data Analysis

1. **Revenue Analysis by Category**

*Purpose: This query identifies which genres are the primary drivers of income. It joins five different tables to link the customer’s payment back to the specific category of the film they rented.*

```sql
SELECT 
    c.name AS Category, 
    SUM(p.amount) AS Total_Revenue
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film_category fc ON i.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
GROUP BY 1
ORDER BY 2 DESC;
```




2. **Operational Efficiency: Late Return Rates**

*Purpose: This query calculates how many films are returned after their allowed rental period. It compares the actual time the customer kept the movie (`DATEDIFF`) against the film’s specific rental_duration policy.*

```sql
SELECT 
    c.name AS Category,
    SUM(CASE WHEN DATEDIFF(r.return_date, r.rental_date) > f.rental_duration THEN 1 ELSE 0 END) AS Late_Returns,
    SUM(CASE WHEN DATEDIFF(r.return_date, r.rental_date) <= f.rental_duration THEN 1 ELSE 0 END) AS On_Time_Returns
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
WHERE r.return_date IS NOT NULL
GROUP BY 1
ORDER BY Late_Returns DESC;
```




3. **Market Reach: Top 10 Countries by Revenue**

*Purpose: This query aggregates total sales based on the customer's location. It travels from the payment table all the way to the country table to show which international markets are most profitable.*

```sql
SELECT 
    co.country AS Country, 
    SUM(p.amount) AS Total_Revenue,
    COUNT(DISTINCT cu.customer_id) AS Total_Customers
FROM payment p
JOIN customer cu ON p.customer_id = cu.customer_id
JOIN address a ON cu.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```


## Operational Dashboard

<img width="1234" height="689" alt="Sakila Rental Analysis" src="https://github.com/user-attachments/assets/91a0b17a-602e-4e52-af18-56d379acf034" />




## Business Insights

- **Revenue Leaders:** Categories like Sports and Sci-Fi are the top revenue generators.

- **Operational Leaks:** A significant portion of inventory is tied up in late returns, particularly in the "Action" category.

- **Market Reach:** The customer base is globally distributed, with high-value clusters identified in specific regions that lack physical store presence.



## Recommendations

- **Policy Adjustment:** Management should Implement stricter late fee structures or shorter rental periods for high-demand categories to increase inventory turnover.

- **Targeted Marketing:** Focus promotional efforts on high-LTV regions identified in the geographic analysis.

- **Inventory Optimization:** Reallocate budget from low-demand "shelf-warmers" to top-performing genres.
