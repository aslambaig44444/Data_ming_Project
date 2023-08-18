# Data_ming_Project
This project aims to provide a comprehensive solution for managing customer-related data in a retail environment using MySQL, a powerful and widely-used open-source relational database management system
use onlineretail;
show databases;
SELECT * FROM onlinretail.onlinretail;
----------- What is the distribution of order values across all customers in the dataset?
SELECT count(c.Quantity),c.CustomerID
FROM onlineretail.`online retail` as c
group by CustomerID
order by count(Quantity) ;
 --   How many unique products has each customer purchased?
SELECT CustomerID, COUNT(DISTINCT Description) AS UniqueProductsCount
FROM onlineretail.`online retail` as c
GROUP BY CustomerID;
 --    Which customers have only made a single purchase from the company?
SELECT CustomerID
FROM onlineretail.`online retail` as c
GROUP BY CustomerID
HAVING COUNT(DISTINCT InvoiceNo) = 1;

--  Which products are most commonly purchased together by customers in the dataset?
SELECT GROUP_CONCAT(DISTINCT Description ORDER BY Description ASC) AS ProductCombination,
       COUNT(*) AS Frequency
FROM onlineretail.`online retail`
GROUP BY InvoiceNo
HAVING COUNT(*) > 1
ORDER BY Frequency DESC
LIMIT 10;



              --  Advance
-- 1.      Customer Segmentation by Purchase Frequency

-- Group customers into segments based on their purchase frequency, such as high, medium, and low frequency customers. This can help you identify your most loyal customers and those who need more attention.
SELECT CustomerID,
       CASE
           WHEN TotalPurchases <= 3 THEN 'Low Frequency'
           WHEN TotalPurchases <= 6 THEN 'Medium Frequency'
           ELSE 'High Frequency'
       END AS PurchaseSegment
FROM (
    SELECT CustomerID, COUNT(DISTINCT InvoiceNo) AS TotalPurchases
    FROM onlineretail.`online retail`
    GROUP BY CustomerID
) AS PurchaseCounts;


-- Average Order Value by Country

-- Calculate the average order value for each country to identify where your most valuable customers are located.

SELECT Country,
       AVG(TotalOrderValue) AS AverageOrderValue
FROM (
    SELECT Country, InvoiceNo, SUM(Quantity * UnitPrice) AS TotalOrderValue
    FROM onlineretail.`online retail` as c
    GROUP BY Country, InvoiceNo
) AS OrderTotalsByInvoice
GROUP BY Country
ORDER BY AverageOrderValue DESC;


-- 3. Customer Churn Analysis
-- Identify customers who haven't made a purchase in a specific period (e.g., last 6 months) to assess churn
SELECT CustomerID
FROM onlineretail.`online retail` as c
WHERE CustomerID IS NOT NULL
GROUP BY CustomerID
HAVING MAX(InvoiceDate) < DATE_SUB(NOW(), INTERVAL 6 MONTH);


-- 4. Product Affinity Analysis
-- Determine which products are often purchased together by calculating the correlation between product purchases.
WITH OrderTotals AS (
    SELECT InvoiceNo, Description, SUM(Quantity * UnitPrice) AS TotalOrderValue
   FROM onlineretail.`online retail` as c
    GROUP BY InvoiceNo, Description
),
ProductOrders AS (
    SELECT InvoiceNo
    FROM OrderTotals
    WHERE Description = 'Product1Description'
),
CoOccurrence AS (
    SELECT o1.InvoiceNo AS Invoice1, o2.InvoiceNo AS Invoice2
    FROM ProductOrders p1
    JOIN OrderTotals o1 ON p1.InvoiceNo = o1.InvoiceNo
    JOIN OrderTotals o2 ON o1.InvoiceNo = o2.InvoiceNo
    WHERE o2.Description = 'Product2Description' AND o1.Description < o2.Description
)
SELECT p1.Description AS Product1,
       p2.Description AS Product2,
       COUNT(DISTINCT c.Invoice1) AS CoOccurrenceCount
FROM CoOccurrence c
JOIN OrderTotals p1 ON c.Invoice1 = p1.InvoiceNo
JOIN OrderTotals p2 ON c.Invoice2 = p2.InvoiceNo
GROUP BY p1.Description, p2.Description
ORDER BY CoOccurrenceCount DESC;

-- 5. Time-based Analysis
-- Explore trends in customer behavior over time, such as monthly or quarterly sales patterns.
SELECT YEAR(InvoiceDate) AS SalesYear,
       QUARTER(InvoiceDate) AS SalesQuarter,
       SUM(TotalPrice) AS TotalSales
FROM (
    SELECT InvoiceDate, SUM(Quantity * UnitPrice) AS TotalPrice
    FROM onlineretail.`online retail` as c
    GROUP BY InvoiceDate
) AS InvoiceTotals
GROUP BY SalesYear, SalesQuarter
ORDER BY SalesYear, SalesQuarter;
