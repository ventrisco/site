+++
date = "2017-03-08T21:58:50-08:00"
author = ""
type = "post"
featured = ""
title = "LSTM Madness/03/Our first model"
linktitle = ""
description = ""
categories = []
featuredalt = ""
featuredpath = ""
+++

# Recap

Let's look at where we're at relative to the tasks we outlined to complete our first model:

1. ~~Get today's returns for our x values.~~
2. ~~Get the next returns for our y values.~~
3. ~~Shape the data for Keras~~
4. Setup, compile and fit the model.
5. Make predictions on our existing data.
6. Plot the predicted returns to the actual returns.

In this post, we'll cover 4 through 6 and have our first model. The code for this post is located [here](https://github.com/ventrisco/lstm-madness/tree/master/01).


# Model time

Let's build our model.

```
number_of_features = train_x.shape[2]

model = Sequential()
model.add(LSTM(1, input_dim=number_of_features))
model.add(Dense(1))
model.compile(loss='mean_squared_error', optimizer='adam')
model.fit(train_x, train_y, nb_epoch=100, batch_size=1, verbose=2)
```

A neural network requires at least three layers. You always need an input layer and an output layer. The layers in between are the hidden layers where the neurons do their work. Read through [this](http://stats.stackexchange.com/questions/181/how-to-choose-the-number-of-hidden-layers-and-nodes-in-a-feedforward-neural-netw) answer on Stack Overflow for a great walkthrough of the layers. 

In our example, as far I can tell, after we instantiate our model with `model = Sequential()` we're adding the input layer and a hidden layer by calling `model.add(LSTM(1, input_dim=number_of_features))`. The first parameter in `LSTM` is the number of neurons for that layer while `input_dim` tells the model how many features we have. Our output layer is called by the `Dense` function which has a output of 1 dimension. 

After our model is set up, we run `fit`, with our X values and Y values. `nb_epoch` is the number of times we we pass over the the data while `batch_size` is the number of samples (X values) the model uses at one time. For the sake of argument, let's say we have 2,000 X values. Using a `batch_size` of 1 means the model will iterate over the X values 2,000 times for each epoch. If our `batch_size` was 200, it would only iterate over the X values 10 times for each epoch.

All of these parameters are referred to as hyperparameters in neural network speak. In machine learning, when you see hyper-anything means it's going to be like looking at the McDonald's menu after the they started serving breakfast all day. More parameters are great but they add the potential for cognitive overload which may ultimately lead to overfitting your data and a bad headache.

# The Fit Bit

```
# make predictions
train_predict = model.predict(train_x)
test_predict = model.predict(test_x)

# Add prediction to y_df_dataframe
y_df_train = y_df[0:train_x.shape[0]]
y_df_train["prediction"] = train_predict
y_df_train.to_sql('lstm_madness_xlf_train_01', engine) 
```

Here we take the trained model and predict Y values with our training data. We then take those results, line them up with the our actual Y values and save that dataframe as a table in our database. We'll use these tables to see how well our trained models can predict the values it was trained with. 

# Plots

For each iteration, I'm going to generate two initial plots. 

1. A scatter plot comparing the predicted return to the actual return.
2. A line plot comparing the cumulative actual returns and the cumulative predicted returns.

My preference is to iterate quickly so I usually take a quick look at these two plots and just jump into another iteration. I'll only start running other reports and plotting those as well if something interesting comes up. 

Let's take a look at what we got. 

![Scatterplot](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/scatter.png)

In an ideal world, what you want to see in this scatterplot is a linear line that goes from the lower left quadrant to the upper right quadrant. This would mean that our trained model perfectly predicted the actual return. We don't live in an ideal world and this scatter plot shows that our model doesn't make very good predictions. It looks like the majority of its predictions are negative. 

Let's take a look at our cumulative return plot. 

![Cumulative Returns](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/cum_ret.png)


Yup, the majority of those predictions are negative. The red line represents the sum of the XLF returns that results in a chart that looks like the actual price chart for the XLF. The financial markets have had some rough spots over the past 18 years and the XLF's actual cumulative returns reflects that. Our predictions on the other hand are consistently negative and looks more like a random negative drift than a series of hard predictions. The spike of activity in the 2009 may be related to better "edges" that the model was able to detect due to increased volatility during that year.

# Conclusion

Our first model was a bust. The predictions of our trained model against the trained data is so lame, it doesn't make any sense to use our trained model on out of sample data. The trained model, however, shows that increased volatility results in more prediction activity. We'll explore how we can use this in our next iteration. 
