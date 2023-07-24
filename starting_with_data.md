Question 1: 
find the total number of unique visitors (fullVisitorID) 

SQL Queries:
SELECT COUNT(DISTINCT(fullvisitorid))
FROM public.allsessions 

Answer: 
14223


Question 2: 
find the total number of unique visitors by referring sites 

SQL Queries:
SELECT COUNT(DISTINCT(visitid))
FROM public.analytics 
WHERE visitnumber > 0


Answer:
148642


Question 3: 
find each unique product viewed by each visitor 

SQL Queries:
SELECT DISTINCT(p.name), ana.visitnumber
FROM public.products p
JOIN public.allsessions a
ON a.productsku = p.sku
JOIN public.analytics ana USING(visitid)
WHERE ana.visitnumber > 0

Answer:



Question 4: 
compute the percentage of visitors to the site that actually makes a purchase


SQL Queries:
SELECT 100*(SELECT COUNT(*)
FROM public.analytics 
WHERE unitssold > 0)::FLOAT/(SELECT COUNT(*)
FROM public.analytics)


Answer:
2.2%


Question 5: 

SQL Queries:

Answer:
