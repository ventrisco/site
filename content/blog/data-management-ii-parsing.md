+++
featuredalt = ""
featuredpath = ""
featured = ""
date = "2017-02-22T18:50:16-08:00"
categories = []
description = ""
author = "M. Ventris"
type = "post"
title = "data management ii parsing"
linktitle = ""

+++

### Data Structure

We've got our data, now what do we do with it? Rather than insert each line into the database separately we're going use Postgresql's `copy from` function to perform a bulk insert. Before we do that we have to write the data to file.

Let's take a look at our data and think about how we want to parse the data and set up our table.

```
Date,Open,High,Low,Close,Volume,Adj Close
2017-02-13,133.080002,133.820007,132.75,133.289993,22926400,133.289993
2017-02-10,132.460007,132.940002,132.050003,132.119995,20065500,132.119995
```

It's obvious that we'll need a table with the columns `date, open, high, low, close, volume, and adj_close`. But what about our ticker symbol. One way to attack the problem is to create a table for each ticker symbol. Although this will work, if we have more than a couple ticker symbols, managing all those tables would be a mess and you would probably have to keep track of your data in the  application layer.

For sanity's sake, we're going to use a single table for our historical data with all of the columns mentioned above and another column labeled `ticker`. This requires us to split the data on each line and add the ticker symbol at the beginning of each line before we write it to disk. In our example, a written line should look like this:

```
AAPL, 2017-02-13,133.080002,133.820007,132.75,133.289993,22926400,133.289993
AAPL, 2017-02-10,132.460007,132.940002,132.050003,132.119995,20065500,132.119995
```

Now that we know what we're trying to do let's take a look at some code.

```
# initialize data.txt
f = open('/tmp/data.txt', 'w')
f.close()

# Parse and write data
for d in data:
    Date, Open, High, Low, Close, Volume, Adj_Close = d.split(',')
    data_string = '%s,%s,%s,%s,%s,%s,%s,%s' % (TICKER, Date, Open, High, Low, Close, Volume, Adj_Close)
    f = open('/tmp/data.txt', 'a')
    f.write(data_string)
    f.close()
```
First we initialize `/tmp/data.txt` so that we have an empty file we can append. If you don't do this, you might have data left over from the last time you ran this script.

For every line `d` in `data`, we split the line on the commas so that each field gets mapped to their corresponding variable. Next, `data_string` represents a new line with the ticker symbol prepended to it. This new line then gets written to `/tmp/data.txt`.

Now that we have our data written to disk let's copy it to our Postgresql cluster.
