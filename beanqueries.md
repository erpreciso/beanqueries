---
layout: post
title: "My beanqueries"
description: ""
#author: "erpreciso"
#author_email: "trepreciso@gmail.com"
tags: []
---

Get all balances in commodities:

~~~sql
SELECT date, account, units(sum(position)) as units
  WHERE 'InvestmentTransaction' in tags
  AND NOT account ~ "Assets:xxx:INGB:Cash|Assets:xxx:xxx:Cash|Expenses|Income|OffsetPot"
  GROUP BY date, account
~~~

Get all balances on a certain date:

~~~sql
balances FROM OPEN ON 2022-01-01 CLOSE ON 2022-01-02 CLEAR WHERE account ~ "INGB"
~~~

List all expenses of the month:

~~~sql
SELECT 
    account, sum(cost(position)) as total, month 
WHERE
    account ~ "Expenses:*" and year = YEAR(today()) and month = MONTH(today()) 
GROUP BY month, account 
ORDER BY total, account DESC
~~~

List units, book and market of my investments:

~~~sql
SELECT 
	account,
    units(sum(position)) AS quantity,
    cost(sum(position)) AS Book,
    convert(units(sum(position)), "EUR") AS Market
WHERE
	account ~ "Assets:xxx:INGB" 
GROUP BY account 
ORDER BY account
~~~

Get summary income and expenses by month:

~~~sql
SELECT
    year, month, root(account, 1) as account, sum(position) as total
  FROM
    date > 2021-01-01 AND date < 2021-12-31
  WHERE
     account ~ "Expenses" OR
     account ~ "Liabilities:Mortgage" OR
     account ~ "Liabilities:Loan" OR
     account ~ "Income"
  GROUP BY year, month, account
  ORDER BY year, month, account
~~~

Render another expenses by month:

~~~sql
SELECT
  MONTH(date) AS month,
  SUM(COST(position)) AS balance
WHERE
   account ~ 'Expenses:' AND
   currency = 'EUR' AND
   YEAR(date) = 2021
GROUP BY 1
ORDER BY 1;
~~~

Get expenses for an account:

~~~sql
SELECT
  date, account, position, balance 
WHERE 
  account ~ 'Landline';
~~~

Get expenses by month for the car accounts:

~~~sql
SELECT year, month, sum(position)
FROM year >= 2017
WHERE account ~ ':car:'
GROUP BY year, month
ORDER BY year, month DESC
~~~

Get all expenses converted in EUR and ready to export to CSV and import in sheets

~~~sql
SELECT year,month, sum(cost(position)), sum(CONVERT(position, 'EUR'))
FROM year >= 2017
WHERE account ~ ':car:'
GROUP BY year, month
ORDER BY year,month desc
~~~

Get car expenses:

~~~sql
SELECT year, month, quarter(date), account, sum(convert(position, 'EUR')) AS val
FROM year >=2015
WHERE account ~ "Expenses:Family:Car|Expenses:Family:Van"
GROUP BY year,month, account, quarter(date)
~~~

Filter on accounts metadata

~~~sql
SELECT getitem(open_meta(account), 'name') as name, value(sum(position)) AS balance
WHERE getitem(open_meta(account), 'report') = 'true'
~~~