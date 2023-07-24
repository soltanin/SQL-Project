What issues will you address by cleaning the data?
**ALL SESSIONS TABLE
-----------------------
1- determine (and remove) columns that more then 50% of them is null

And time1 and timeonsite columns need datatype change


2-
have (no set) values:
country, city, v2productcategory columns 

have null values:    
timeonsite: fill null values with time values of time column
currencycode columns : fill missing values with 'USD' as other observations
pagetitle  : has one null value, which is filled with mode

         
**ANALYTICS TABLE
-----------------
1- determine (and remove) columns that more then 50% of them is null; and remove socialengagementtype which all of its value is 'Not Socially Engaged'
2- pageviews has null values, we remove them because they are a low percent of data

**PRODUCT TABLE
--------------
sentimentscore AND sentimentmagnitude have one missing value, we remove it.

**SALES REPORT TABLE
---------------------
ratio have some missing values because of the zero values for stock level. Then we fill null values with this expression: 'stock level is zero'

At the end create/determine primary and forien keys for each table and load cleaned data to new tables
*******************************************************************************************************************************************************************
Queries:
Below, provide the SQL queries you used to clean your data.

**ALL SESSIONS TABLE
--------------------
1- SELECT (SELECT COUNT(*) 
FROM public.allsessions
WHERE totaltransactionrevenue IS NULL)::FLOAT/(SELECT COUNT(*) FROM public.allsessions) output = 99% is null ==> remove totaltransactionrevenue column

SELECT (SELECT COUNT(*) 
FROM public.allsessions
WHERE transactions IS NULL)::FLOAT/(SELECT COUNT(*) FROM public.allsessions) output = 99% is null ==> remove transactions column


With similar queries, we decide to remove below columns:
sessionqualitydim (92% null), productrefundamount (100% null!), productquantity(99% null), productrevenue (99% is null), itemquantity (100%), itemrevenue(100%), 
transactionrevenue(99%), transactionid(99%), searchkeyword(100%), ecommerceactionoption(99%), productvariant (99% not set)

SELECT fullvisitorid, channelgrouping, TO_TIMESTAMP(time1) AS time, country,
city, COALESCE(TO_TIMESTAMP(timeonsite)::time, TO_TIMESTAMP(time1)::time) AS timeonsite, 
pageviews, date, visitid, type1, productprice, productsku, v2productname, v2productcategory, 
COALESCE (currencycode, 'USD') AS currencycode, COALESCE(pagetitle, 'YouTube | Shop by Brand | Google Merchandise Store') AS pagetitle, 
pagepathlevel1, ecommerceactiontype, ecommerceactionstep
FROM public.allsessions

determine mode! == > 
SELECT COUNT(*), pagetitle 
FROM public.allsessions
GROUP BY pagetitle 
ORDER BY COUNT(*) DESC


**ANALYTICS TABLE
----------------
1- SELECT  fullvisitorid, visitid, visitnumber, TO_TIMESTAMP(visitstarttime) AS visitstarttime,
channelgrouping, pageviews, unitprice::float/1000000 AS unitprice
FROM public.analytics
WHERE pageviews IS NOT NULL

**PRODUCT TABEL
--------------
SELECT * 
FROM public.products
WHERE (sentimentscore IS NOT NULL) AND (sentimentmagnitude IS NOT NULL)


**SALES REPORT TABLE
-----------------
SELECT productsku, totalordered, name, stocklevel, TO_TIMESTAMP(restockingleadtime)::time
AS restockingleadtime, sentimentscore, sentimentmagnitude, COALESCE(ratio::text, 'stock level is zero') AS ratio
FROM public.salesreport

**LOAD cleaned data

**CREATE TABLE public.fullvisitorkey
(
    fullvisitorid numeric NOT NULL,
    PRIMARY KEY (fullvisitorid)
);

ALTER TABLE IF EXISTS public.fullvisitorkey
    OWNER to postgres;
	
INSERT INTO fullvisitorkey
SELECT DISTINCT(fullvisitorid) FROM public.allsessions
UNION 
SELECT DISTINCT(fullvisitorid) FROM public.analytics
------------------------------
**CREATE TABLE public.visitkey
(
    visitid bigint NOT NULL,
    PRIMARY KEY (visitid)
);

ALTER TABLE IF EXISTS public.visitkey
    OWNER to postgres;
	
INSERT INTO visitkey
SELECT  DISTINCT(visitid) FROM public.allsessions
UNION
SELECT  DISTINCT(visitid) FROM public.analytics
-------------------
**CREATE TABLE public.skukey
(
    sku character varying NOT NULL,
    PRIMARY KEY (sku)
);

ALTER TABLE IF EXISTS public.skukey
    OWNER to postgres;
	
INSERT INTO skukey
SELECT  DISTINCT(productsku) FROM public.allsessions
UNION
SELECT  DISTINCT(sku) FROM public.products
UNION 
SELECT  DISTINCT(productsku) FROM public.salesbysku
UNION
SELECT  DISTINCT(productsku) FROM public.salesreport
----------------------
**CREATE TABLE public.newallsession
(
    fullvisitorid numeric, channelgrouping character varying, 	time timestamp,
	country character varying,	city character varying, timeonsite time, pageviews int, date date,
	visitid int, type character varying, productprice bigint, productsku character varying,
	v2productname character varying, v2productcategory character varying,
	currencycode character varying, pagetitle character varying, pagepathlevel1 character varying,
	ecommerceactiontype int, ecommerceactionstep int,
	FOREIGN KEY (fullvisitorid) REFERENCES fullvisitorkey(fullvisitorid),
	FOREIGN KEY (visitid) REFERENCES visitkey(visitid),
	FOREIGN KEY (productsku) REFERENCES skukey(sku)
);

ALTER TABLE IF EXISTS public.newallsession
    OWNER to postgres;
	
INSERT INTO newallsession
SELECT fullvisitorid, 
channelgrouping, TO_TIMESTAMP(time1) AS time, country,
city, COALESCE(TO_TIMESTAMP(timeonsite)::time, TO_TIMESTAMP(time1)::time) AS timeonsite, 
pageviews, date, visitid, 
type1, productprice, productsku, 
v2productname, v2productcategory, 
COALESCE (currencycode, 'USD') AS currencycode, COALESCE(pagetitle, 'YouTube | Shop by Brand | Google Merchandise Store') AS pagetitle, 
pagepathlevel1, ecommerceactiontype, ecommerceactionstep

FROM public.allsessions
----------------------
**CREATE TABLE public.newanalytics
(
    fullvisitorid numeric, visitid bigint, visitnumber int, visitstarttime timestamp,
    channelgrouping character varying, pageviews int, unitprice float,
    FOREIGN KEY (fullvisitorid) REFERENCES fullvisitorkey(fullvisitorid),
	FOREIGN KEY (visitid) REFERENCES visitkey(visitid)
);

ALTER TABLE IF EXISTS public.newanalytics
    OWNER to postgres;


INSERT INTO newanalytics
SELECT  fullvisitorid, visitid, visitnumber, TO_TIMESTAMP(visitstarttime) AS visitstarttime,
channelgrouping, pageviews, unitprice::float/1000000 AS unitprice
FROM public.analytics
WHERE pageviews IS NOT NULL

---------------------------
** CREATE TABLE public.newproducts
(
    sku character varying NOT NULL, name character varying,	orderedquantity int,
	stocklevel int, restockingleadtime int, sentimentscore float, 
	sentimentmagnitude float,
    FOREIGN KEY (sku) REFERENCES skukey(sku)
);

ALTER TABLE IF EXISTS public.newproducts
    OWNER to postgres;


INSERT INTO newproducts
SELECT * 
FROM public.products
WHERE (sentimentscore IS NOT NULL) AND (sentimentmagnitude IS NOT NULL)
---------------------------------------
**CREATE TABLE public.newsalesbysku
(
    productsku character varying, totalordered int,
    FOREIGN KEY (productsku) REFERENCES skukey(sku)
);

ALTER TABLE IF EXISTS public.newsalesbysku
    OWNER to postgres;


INSERT INTO newsalesbysku
SELECT * FROM public.salesbysku
-----------------------------------------
**CREATE TABLE public.newsalesreport
(
    productsku character varying, totalordered int,	name character varying, 
	stocklevel int, restockingleadtime time, sentimentscore float,
    sentimentmagnitude float, ratio text,
    FOREIGN KEY (productsku) REFERENCES skukey(sku)
);

ALTER TABLE IF EXISTS public.newsalesreport
    OWNER to postgres;


INSERT INTO newsalesreport
SELECT productsku, totalordered, name, stocklevel, TO_TIMESTAMP(restockingleadtime)::time
AS restockingleadtime, sentimentscore, sentimentmagnitude, COALESCE(ratio::text, 'stock level is zero') AS ratio
FROM public.salesreport