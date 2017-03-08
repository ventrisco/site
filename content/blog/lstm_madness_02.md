+++
date = "2017-03-07T13:33:49-08:00"
description = ""
categories = []
type = "post"
featuredalt = ""
title = "LSTM Madness/02/Data Shaping"
author = "M. Ventris"
featuredpath = ""
featured = ""
linktitle = ""

+++

# Tomorrow never knows

For our first neural network, we'll try to answer the most obvious question:

> Can today's return predict tomorrow's return?

Let's break this down to actionable tasks:

1. We'll need to get today's returns for our x values.
2. We'll need to get the next returns for our y values.
3. We'll need to shape the data for Keras
4. We'll need to setup, compile and fit the model.
5. We can then use the model to make predictions on our existing data.
6. We'll plot the predicted returns to the actual returns.

We're going to ignore applying our model to test data while we try to figure out what all the levers do in the neural network cockpit.

# Data Shaping

> You can skip to the next post if you want to jump right into the LSTM model.

Many machine learning examples completely ignore the process of shaping data which is the process of take raw data and shaping it so it's in a format that can be read by our machine learning tools. Often times, this takes much longer than actually setting up and running the machine learning process. Since every data source is slightly different this process includes a lot of trial and error.

After you've imported your dependencies and connected to your database,
let's query our database and get the XLF returns. We're going use `pandas` for this so our results will be returned in a `pandas dataframe`.

```
x_query = """
SELECT
  dt
, ticker
, ret
FROM xlf_returns
WHERE dt > '1998-12-22'
AND dt < '2017-02-28'
ORDER BY dt
"""

x_df = pd.read_sql_query(x_query, con=conn)
```

If you type `x_df` on the command line you should have something that looks like this:

```
>>> x_df
              dt ticker       ret
0     1998-12-23    xlf  0.014745
1     1998-12-24    xlf  0.006605
2     1998-12-28    xlf -0.013123
3     1998-12-29    xlf  0.010638
4     1998-12-30    xlf -0.003947
...          ...    ...       ...
4568  2017-02-21    xlf  0.004904
4569  2017-02-22    xlf  0.000813
4570  2017-02-23    xlf  0.000000
4571  2017-02-24    xlf -0.007720
4572  2017-02-27    xlf  0.005323

[4573 rows x 3 columns]
```

Before we go on, it's helpful to see what our dataset ultimately needs to look like.

```
>>> dataset_x
array([[ 0.01474535],
       [ 0.00660498],
       [-0.01312336],
       ...,
       [ 0.        ],
       [-0.00772048],
       [ 0.00532346]], dtype=float32)
```

We need to take our dataframe and shape it into a `numpy.array` of datatype `float32`. This seems pretty straightforward, right? Nope. My first attempt at converting this was to just take `x_df.ret` and convert it to `floast32`. I ended up with this:

```
test = x_df["ret"].astype('float32')

>>> test
0       0.014745
1       0.006605
2      -0.013123
3       0.010638
4      -0.003947
5      -0.009247
6       0.000000
          ...   
4569    0.000813
4570    0.000000
4571   -0.007720
4572    0.005323
Name: ret, dtype: float32
```

It got the data type right, `float32`, but it's just a column of numbers. It's not even an array. We should be able to easily convert that to an array.

```
test_array = numpy.array(test)

>>> test_array
array([ 0.01474535,  0.00660498, -0.01312336, ...,  0.,
       -0.00772048,  0.00532346], dtype=float32)
```

Now we have an array with the right datatype but if you go back and look at what the final dataset needs to look like we actually need an array of arrays. This makes sense when you consider that we would need to be able to use array of past returns as X values to feed into our model. At this point, I would create a `for loop` to create an array for each value in the array but I inadvertantly ran into another solution. 

```
x_df_wide = x_df.pivot(index="dt", columns="ticker", values="ret")


>>> x_df_wide
ticker           xlf
dt                  
1998-12-23  0.014745
1998-12-24  0.006605
1998-12-28 -0.013123
1998-12-29  0.010638

dataset_x = x_df_wide.values.astype('float32')

>>> dataset_x
array([[ 0.01474535],
       [ 0.00660498],
       [-0.01312336],
       ..., 
       [ 0.        ],
       [-0.00772048],
       [ 0.00532346]], dtype=float32)
```

Here we take the tall dataframe and make it wide by pivoting the dataframe. With only one ticker, we end up with a dataframe that has the dates as the index and XLF as the column with its returns as row values. If we had more than one ticker, we would have a separate column for each header. For some reason, when you convert wide dataframe's values to a numpy array you end up with the shape we need for our model. We also get the added bonus of having figured out how we use additional pull in and shape data with more than one feature. Here's a quick summary of where we're at.

```
# We're using date ranges to avoid `null` values.
# Get x data

x_query = """
SELECT
  dt
, ticker
, ret
FROM xlf_returns
WHERE dt > '1998-12-22'
AND dt < '2017-02-28'
ORDER BY dt
"""

x_df = pd.read_sql_query(x_query, con=conn)
x_df_wide = x_df.pivot(index="dt", columns="ticker", values="ret")

# Get y data

y_query = """
SELECT
  dt
, ticker
, ret
FROM xlf_next_day_returns
WHERE dt > '1998-12-22'
AND dt < '2017-02-28'
ORDER BY dt;
"""

y_df = pd.read_sql_query(y_query, con=conn)
y_df_wide = y_df.pivot(index="dt", columns="ticker", values="ret")

dataset_x = x_df_wide.values.astype('float32')
dataset_y = y_df_wide.values.astype('float32')
```

The next bit of code splits our dataset so that 67% will be used to train our network while the remainder will be used to test its predictions. Confusingly, you also need to predict against your training data to see how well the model fits your training data. I'll cover more on this later. 

```
train_size = int(len(dataset_x) * 0.67)
test_size = len(dataset_x) - train_size
stage_train_x, stage_test_x = dataset_x[0:train_size], dataset_x[train_size:len(dataset_x)]
train_y,  test_y = dataset_y[0:train_size], dataset_y[train_size:len(dataset_y)]
```

The last thing we have to do is add the time step to the shape of the X value datasets. A numpy array is an object with the property `shape`. `stage_train_x.shape` returns `(3063, 1)` which reflects the number of objects in our array, 3063, and the number of items in each object, 1. That's great but the LSTM model also needs a value for the number of time steps or the lookback period of our data. Since we're only using one day's worth of data, our time step value is 1 and so we'll add that to the `shape` property.

```
train_x = numpy.reshape(stage_train_x, (stage_train_x.shape[0], 1, stage_train_x.shape[1]))
test_x = numpy.reshape(stage_test_x, (stage_test_x.shape[0], 1, stage_test_x.shape[1]))
```

We only need to do this for the X values. Now that we have the right shape, we can finally start building our model. 

