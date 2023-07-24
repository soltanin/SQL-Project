Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
SELECT country,city, totaltransactionrevenue
FROM public.allsessions
WHERE totaltransactionrevenue IS NOT null
ORDER BY totaltransactionrevenue DESC



Answer:
United States, Atlanta



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

SELECT a.country, a.city, AVG(p.orderedquantity) AS avg_orderedquantity
FROM public.newallsession a
JOIN public.newproducts p
ON a.productsku = p.sku
GROUP BY a.country, a.city
ORDER BY avg_orderedquantity DESC

Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
SELECT a.country, a.city, a.v2productcategory, count(a.v2productcategory)
FROM public.newallsession a
JOIN public.newproducts p
ON a.productsku = p.sku
WHERE orderedquantity > 0
GROUP BY a.country, a.city, a.v2productcategory
ORDER BY count(a.v2productcategory) DESC


Answer:
Most of the orders are from United State and thier city is not available in demo data set




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
SELECT a.country, a.city, a.v2productcategory, count(a.v2productcategory)
FROM public.newallsession a
JOIN public.newproducts p
ON a.productsku = p.sku
WHERE orderedquantity > 0
GROUP BY a.country, a.city, a.v2productcategory
LIMIT 1



Answer:
Home/Bags/ is top selling product from United State.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
SELECT a.country, a.city, SUM(ana.revenue)
FROM public.allsessions a
JOIN public.analytics ana USING(fullvisitorid)
GROUP BY a.country, a.city
HAVING SUM(ana.revenue) IS NOT NULL
ORDER BY SUM(ana.revenue) DESC


Answer:
United state has the most revenue






