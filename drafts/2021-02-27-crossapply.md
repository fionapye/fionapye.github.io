---
layout: post
title: PIVOT() 
subtitle: Pivot functions, from basic to dynamic
gh-repo: 
gh-badge: [star, fork, follow]
tags: [SQL, SQLServer, T-SQL, Pivot]
comments: true
toc: true
---

Pivot is used to turn a long table into a wide table, this can be used for summaries or a different display format for data.

```
## Simple Pivot
## Pivot with Multiple Aggregates
## Multiple Pivot
## Unpivot


## Pivot with Dynamic SQL

When the resultant table will contain lots of columns, its best to use dynamic SQL to work out the column headers so that they don't have to be written out explicitly.

To find out more about [dynamic SQL](comingsoon) check out the post (coming soon).

Example source: the code is written by me, but the example data/scenario comes from this [stackoverflow question](https://stackoverflow.com/questions/20676984/how-to-pivot-how-to-convert-multiple-rows-into-one-row-with-multiple-columns/66393505#66393505)

**Initial Data**

The data at the start looks like this, two tables relating to clients and products in long form.

_Clients_

| ClientID | Name | 
| :------ | :--- | 
| 1 | Name1 |
| 2 | Name2 |

_Products_

| ProductID | ClientID | Product |
| :------ |:--- | :--- | 
| 1 | 1 | SomeproductA |
| 2 | 1 | SomeproductB |
| 3 | 1 | SomeproductA |
| 4 | 2 | SomeproductC |
| 5 | 2 | SomeproductD |
| 6 | 2 | SomeproductA |

**Result**

Here is the pivoted (wide) table that is created with the dynamic pivot shown in the code block below.

| ClientID | ClientName | Product1 | Product2 | Product3 |
| :------ |:--- | :--- | :--- | :--- | 
| 1 | Name1 | SomeproductA | SomeproductA | SomeproductB |
| 2 | Name2 | SomeproductA | SomeproductC | SomeproductD |


**Code**

{% highlight SQL linenos %}
-- variable tables to store data
DECLARE @Clients TABLE(ClientID int, 
                ClientName nvarchar(10))

DECLARE @Products TABLE(ProductID int, 
                    ClientID int, 
                    Product nvarchar(15))

-- populate the variable tables with sample data
INSERT INTO @Clients 
VALUES (1, 'Name1'),
    (2, 'Name2')

INSERT INTO @Products 
VALUES (1, 1, 'SomeproductA'),
    (2, 1, 'SomeproductB'),
    (3, 1, 'SomeproductA'),
    (4, 2, 'SomeproductC'),
    (5, 2, 'SomeproductD'),
    (6, 2, 'SomeproductA')

-- display the tables to check
SELECT * FROM @Clients
SELECT * FROM @Products

-- join the two tables and generate a column with rows which will become the new 
-- column names (Product_col) which gives a number to each product per client
SELECT c.ClientID, 
    c.ClientName, 
    p.ProductID, 
    p.Product,
    CONCAT('Product', ROW_NUMBER() 
        OVER(PARTITION BY c.ClientID ORDER BY p.Product ASC))  AS Product_col
INTO #Client_Products
FROM @Products p 
LEFT JOIN @Clients c ON c.ClientID = p.ClientID

-- view the joined data and future column headings
SELECT * FROM #Client_Products

-- setup for the pivot, declare the variables to contain the column names for pivoted 
-- rows and the query string
DECLARE @cols1 AS NVARCHAR(MAX),
    @query  AS NVARCHAR(MAX);

-- column name list for products
SET @cols1 = STUFF((SELECT distinct ',' + QUOTENAME(Product_col) 
        FROM #Client_Products
        FOR XML PATH(''), TYPE
        ).value('.', 'NVARCHAR(MAX)') 
    ,1,1,'')

SELECT @cols1  -- view the future column names

-- generate query variable string

-- the top select is all the columns you want to actually see as the result
-- The inner query needs the columns you want to see in the result, and the columns 
-- you are pivoting with. The pivot needs to select the value you want to go into the 
-- new columns (MAX()) and the values that will become the column names (FOR x IN())
SET @query = 'SELECT ClientID, 
            ClientName,'
                + @cols1 +' 
            FROM
            (
                SELECT ClientID,
                    ClientName,
                    Product_col,
                    Product
                FROM #Client_Products
           ) x
         PIVOT 
        (
            MAX(Product)
            FOR Product_col IN (' + @cols1 + ')
        ) p'


EXECUTE(@query) -- execute the dynamic sql

DROP TABLE #Client_Products

{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.
