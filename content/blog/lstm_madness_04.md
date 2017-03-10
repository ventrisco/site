+++
featuredalt = ""
linktitle = ""
categories = []
author = ""
date = "2017-03-09T08:52:59-08:00"
title = "LSTM Madness/04/First Iteration"
featuredpath = ""
featured = ""
description = ""
type = "post"

+++

# Hyper Time

So our initial model was a bust. You should embrace that because in research knowing what doesn't work is just as important as figuring out what does. Let's start messing with some hyperparameters.

The first thing that I can think of tweaking is the number of epochs. I'm going to increase epochs from 100 to 1000. We run the risk of overfitting the model but it's better than a model that doesn't fit all. 

Since we're going to increase our epochs to 1000, I'm going to increase our batch size to 32 in order to decrease the run time. Our initial model ran 100 epochs with a batch size of 1. It took about 14 seconds to run each epoch on my machine. That's because it's going only fitting 1 data point for each epoch. Why 32? It's an arbitrary number based on some examples on Keras' site, don't read anything into it.

# That's Interesting

Here's the cumulative return of our first model.

![Cumulative Return](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/01/cum_ret.png)

Here's the scatterplot of our predicted returns vs the actual returns for our train data. 

![Scatter](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/01/scatter.png)

This is much better than our first model since it actually "fits" a linear line to some large trends. However, it's odd that it catches those trend changes pretty late even on the train data. The scatterplot is interesting in that it seems as if the larger predictions are negative which is odd when you consider that down markets are volatile to the downside and upside.

Before we move one, let's take a look at the profit and loss.

![Cumulative Profit](https://raw.githubusercontent.com/ventrisco/lstm-madness/master/01/01/cum_profit.png)

Given that this ignores trading costs, it would be like Chinese Water torture trading this model from 2003 to 2008. Since the profits are based on our training data we shouldn't get too excited but a 250% return isn't terrible.

Let's look at a couple iterations of this next to see if we can better our returns. 