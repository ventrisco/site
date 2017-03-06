+++
date = "2017-02-22T18:50:26-08:00"
featured = ""
linktitle = ""
title = "data management iii database"
author = ""
type = "post"
featuredalt = ""
featuredpath = ""
description = ""
categories = []
draft=false
+++

### COPY THAT

The fastest way to get your data into Postgresql is to use the `COPY` command. The `COPY` command is like a batch insert but much faster and you should use every time you find yourself performed serial inserts.

The `COPY` command requires the number of columns in the data to match the number of columns in the table that you're copying to. So, let's create a table for a our data.

```
CREATE TABLE yahoo_data (
  ticker TEXT
, dt DATE
, open NUMERIC
, high NUMERIC
, low NUMERIC
, close NUMERIC
, volume NUMERIC
, adj_close NUMERIC
);
```

Here we just create a table with column names that mirror the header in our csv file. Once you have this table set up, all you need to do is copy your data to the table with the following command:

```
COPY yahoo_data FROM '/tmp/data.txt' WITH DELIMITER ',' CSV HEADER;
```

Here we use `COPY FROM` with the `WITH DELIMITER ','` option that tells Postgresql that our data is comma delimited. The `CSV HEADER` option tells Postgresql to ignore the header. You can ignore the header while writing the data to disk but I found that it's easier to use the `CSV HEADER` option instead.

That's it! You know have stock data that you can query and analyze to your heart's content. If you query your data you should have something that looks like this:

```
historical_stock_data=# select * from yahoo_data limit 10;
 ticker |     dt     |    open    |    high    |    low     |   close    |  volume  | adj_close  
--------+------------+------------+------------+------------+------------+----------+------------
 AAPL   | 2017-02-17 | 135.100006 | 135.830002 | 135.100006 | 135.720001 | 22084500 | 135.720001
 AAPL   | 2017-02-16 | 135.669998 | 135.899994 | 134.839996 | 135.350006 | 22118000 | 135.350006
 AAPL   | 2017-02-15 | 135.520004 | 136.270004 | 134.619995 | 135.509995 | 35501600 | 135.509995
 AAPL   | 2017-02-14 | 133.470001 | 135.089996 |     133.25 | 135.020004 | 32815500 | 135.020004
 AAPL   | 2017-02-13 | 133.080002 | 133.820007 |     132.75 | 133.289993 | 23035400 | 133.289993
 AAPL   | 2017-02-10 | 132.460007 | 132.940002 | 132.050003 | 132.119995 | 20065500 | 132.119995
 AAPL   | 2017-02-09 | 131.649994 | 132.449997 | 131.119995 | 132.419998 | 28349900 | 132.419998
 AAPL   | 2017-02-08 | 131.350006 | 132.220001 | 131.220001 | 132.039993 | 23004100 | 131.469994
 AAPL   | 2017-02-07 | 130.539993 | 132.089996 | 130.449997 | 131.529999 | 38183800 | 130.962201
 AAPL   | 2017-02-06 | 129.130005 |     130.50 | 128.899994 | 130.289993 | 26845900 | 129.727549
(10 rows)
```

### CONCLUSION

Although getting data from different service providers will require you to write different adapters and probably in a different programming language, the basic premise will always be the same: fetch, parse, upload. You could just download a slug of data every time you need it but I found that maintaining the price data in a database makes it easier to deal with when backtesting and it facilitates working with larger data sets. 
