What are your risk areas? Identify and describe them.

I deleted columns that more then 50% (achtually 90%) of their entry was null, while I could calculate them
based on other values. I did not have enogh time to do this.

Below, I provided the query for filling revenue column from analytics table. 
unitssold and unitprice columns can be filled in a similar way. 

In addition, we can fill totaltransactionrevenue, transactions, productrefundamount,
productquantity, productrevenue, itemquantity, itemrevenue and transactionrevenue. 

The remaining columns (that have 90% null values) can be deleted. They are: transactionid, searchkeyword, ecommerceactionoption, sessionqualitydim.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

 **
WITH TABLE1 AS (SELECT DISTINCT(visitid) AS ID, unitssold, unitprice, revenue
				FROM public.analytics
				WHERE (revenue IS NULL) AND (unitssold IS NOT NULL) 
				AND (unitprice IS NOT NULL)
				),

TABLE2 AS(SELECT DISTINCT(visitid) AS ID, unitssold, unitprice, revenue,
				(revenue::float/1000000)-(unitssold *(unitprice::float/1000000)) AS id_remain
				FROM public.analytics
				WHERE (revenue IS NOT NULL) AND (unitssold IS NOT NULL)
				AND (unitprice IS NOT NULL)
),



TABLE3 AS(SELECT ID, t1.unitssold, t1.unitprice, 
COALESCE(t1.revenue, t1.unitssold*t1.unitprice+t2.id_remain) AS revenue
FROM TABLE1 t1
JOIN TABLE2 t2 USING(ID))

SELECT COALESCE(t2.revenue,t3.revenue,0) AS revenue
FROM TABLE2 t2
JOIN TABLE3 t3 USING(ID)

