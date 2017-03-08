+++
date = "2017-03-07T21:58:50-08:00"
author = ""
type = "post"
featured = ""
title = "LSTM MADNESS/03/Our first model"
linktitle = ""
description = ""
categories = []
featuredalt = ""
featuredpath = ""
draft="true"
+++

# Recap

Let's look at where we're at relative to the tasks we outlined to complete our first model:

1. ~~Get today's returns for our x values.~~
2. ~~Get the next returns for our y values.~~
3. ~~Shape the data for Keras~~
4. Setup, compile and fit the model.
5. Make predictions on our existing data.
6. Plot the predicted returns to the actual returns.

In this post, we'll cover 4 through 6 and have our first model.


# Model time

Let's build out model.

```
number_of_features = train_x.shape[2]

model = Sequential()
model.add(LSTM(1, input_dim=number_of_features))
model.add(Dense(1))
model.compile(loss='mean_squared_error', optimizer='adam')
model.fit(train_x, train_y, nb_epoch=100, batch_size=1, verbose=2)
```

The `number_of_features` is 1 since we're only using XLF returns as our X value. The `1` in `LSTM(1, ...)` is the number of neurons in that layer. 

# Plots

Let's take a look at the plots.

![Scatterplot](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/scatter.png)



![Cumulative Returns](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/cum_ret.png)



