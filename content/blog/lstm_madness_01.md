+++
type = "post"
author = "M. Ventris"
date = "2017-03-06T21:27:23-08:00"
title = "LSTM Madness/01/Data Prep"
description = ""
categories = []
featuredalt = ""
featuredpath = ""
featured = ""
linktitle = ""
+++

# Yahoo!

Look at the previous posts about how to get data from Yahoo and use that to create a script to get the historical data from the XLF and each of its components. Don’t know the XLF components by heart? You can find them right here.

If that’s too much for you, you can just run `python main.py` in `/get_yahoo_data` of this repo. You’re going to have to run it twice, once for the components and once for just the XLF price series. Save them to different buffers because we’re going to copy them to different tables.

# In the beginning

So that we’re on the same page, let’s create a new user and database.
```
CREATE USER ventris_admin WITH PASSWORD 'X4NAdu';
ALTER ROLE ventris_admin WITH SUPERUSER;
CREATE DATABASE ventris WITH OWNER ventris_admin;
```
Now you should be able to login into your database like so:
```
psql -d ventris -U ventris_admin -h 127.0.0.1
```
If you followed the instructions above, you should have two files, one with just the XLF price data and the other with the price data for all of its components. Let’s create two tables to copy the data into.
```
DROP TABLE IF EXISTS xlf_components_data;
CREATE TABLE xlf_components_data (
  ticker text
, dt date
, open numeric
, high numeric
, low numeric
, close numeric
, volume numeric
, adj_close numeric
);

COPY xlf_components_data FROM '<YOUR FILEPATH HERE>' WITH DELIMITER ',';

DROP TABLE IF EXISTS xlf_etf_data;
CREATE TABLE xlf_etf_data (
  ticker text
, dt date
, open numeric
, high numeric
, low numeric
, close numeric
, volume numeric
, adj_close numeric
);

COPY xlf_etf_data FROM '<YOUR XLF FILEPATH HERE>' WITH DELIMITER ',';
Calculate Returns
```
Now that we have our price data loaded let’s derive a table of returns for each of these tables. You calculate the return by taking today's close - yesterday's close and dividing the result by yesterday's close. Calculating the returns kills two birds with one stone. It normalizes our data and provides us with a table to help us calculate how much money we made.
```
-- Create returns based on the days's close
DROP TABLE IF EXISTS xlf_components_returns;
CREATE TABLE xlf_components_returns AS
SELECT
  dt
, ticker
, CASE
    WHEN LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt) > 0
      THEN (close - LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt)) / LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt)
    ELSE NULL END AS ret
FROM xlf_components_data;

-- Create returns of xlf
DROP TABLE IF EXISTS xlf_returns;
CREATE TABLE xlf_returns AS
SELECT
  dt
, ticker
, CASE
    WHEN LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt) > 0
      THEN (close - LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt)) / LAG(close, 1) OVER (PARTITION BY ticker ORDER BY dt)
    ELSE NULL END AS ret
FROM xlf_etf_data;
```
This should be pretty straightforward. If Postgresql’s window functions are new to you, you should stop and read up on them. Window functions are indispensable for backtests and is really the most under appreciated feature of Postgresql.

The last thing we have to do is create a table with XLF’s next day’s returns. This will be the Y value for fitting our models and is just like our returns table except the returns are offset by one day. For example, our XLF returns table looks like this:
```
dt     | ticker |           ret           
------------+--------+-------------------------
1998-12-22 | xlf    |                        
1998-12-23 | xlf    |  0.01474535247145115037
1998-12-24 | xlf    |  0.00660497782213908891
1998-12-28 | xlf    | -0.01312336068227701268
```
While our xlf_next_day_returns table looks like this:
```
dt     | ticker |           ret           
------------+--------+-------------------------
1998-12-22 | xlf    |  0.01474535247145115037
1998-12-23 | xlf    |  0.00660497782213908891
1998-12-24 | xlf    | -0.01312336068227701268
1998-12-28 | xlf    |  0.01063829877772755555
```
We could create these returns when we do our SQL calls but I find that it’s cleaner to just create a table with those returns:
```
DROP TABLE IF EXISTS xlf_next_day_returns;
CREATE TABLE xlf_next_day_returns AS
SELECT
  dt
, ticker
, CASE
    WHEN LEAD(close, 1) OVER (PARTITION BY ticker ORDER BY dt) > 0
      THEN (LEAD(close, 1) OVER (PARTITION BY ticker ORDER BY dt) - close) / close
    ELSE NULL END AS ret
FROM xlf_etf_data;
```
Now that you can follow along at home, let’s build our first neural network!
