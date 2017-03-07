+++
linktitle = ""
date = "2017-02-22T18:50:07-08:00"
description = ""
type = "post"
featuredalt = ""
featured = ""
featuredpath = ""
title = "data management i fetching"
categories = []
author = "M. Ventris"

+++

## Fetching Data

Yahoo finance has always been a great resource for free financial information. It was really the one and only option until Google finance came along and unlike other parts of Yahoo, management didn't screw up Yahoo Finance as much as the rest of the site.

Yahoo finance has always made their historical financial data available for download in csv format and it's a breeze to download.

In Python, here's how you fetch data from Yahoo Finance for a stock like Apple (ticker: AAPL).

```
import urllib

base_url = "http://ichart.finance.yahoo.com/table.csv?s=%s"
response = urllib.urlopen(base_url % 'AAPL')
data = response.readlines()
print data
```

That's it! If you run this in the Jupyter Notebook, you should see something like this:

```
['Date,Open,High,Low,Close,Volume,Adj Close\n',
'2017-02-17,135.100006,135.830002,135.100006,135.720001,22084500,135.720001\n',
'2017-02-16,135.669998,135.899994,134.839996,135.350006,22118000,135.350006\n',
'2017-02-15,135.520004,136.270004,134.619995,135.509995,35501600,135.509995\n',
'2017-02-14,133.470001,135.089996,133.25,135.020004,32815500,135.020004\n',
'2017-02-13,133.080002,133.820007,132.75,133.289993,23035400,133.289993\n',
'2017-02-10,132.460007,132.940002,132.050003,132.119995,20065500,132.119995\n',
'2017-02-09,131.649994,132.449997,131.119995,132.419998,2834990
...
```

`urllib` allows you to open arbitrary network resources. Here we build a url by using the Yahoo Finance url, pass in a ticker symbol and user `urllib.urlopen` to download the `csv` historical price data for Apple. We use `readlines`, which give us a list of "lines" that we can parse through.

## OHLC

I'll go through this quickly for those who are new to financial data. This comma delimited file is split into columns with headers Date, Open, High, Low, Close, Volume, Adj Close. This data structure is commonly referred to as OHLC, which is short for open, high, low, close. Here's a quick summary of each column.

**Date:** The date of the financial data in the corresponding row

**Open:** This is the first price at which the security was traded at 9:30AM Eastern Standard Time.

**High:** This is the highest price that the security traded at during the trading day.

**Low:** This is the lowest price that the security traded at during the trading day.

**Volume:** This is the total amount of shares that exchanged hands on the date.

**Adj Close:** If there is an adjustment in reconciling the last trade, it would show up here. This doesn't happen as much as it used it.

This data structure is pervasive in financial data so get comfortable with it. Let's move on to parsing!
