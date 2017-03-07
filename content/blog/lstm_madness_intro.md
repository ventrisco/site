+++
title = "LSTM Madness/00/Intro"
featuredalt = ""
featured = ""
linktitle = ""
description = ""
author = ""
type = "post"
date = "2017-03-06T10:06:01-08:00"
featuredpath = ""
categories = []
+++

# Jumping right in

My original intent with this site was to gradually introduce tools and techniques to apply to quantitative finance with open source tools.  I'm still doing the introducing but dropping the gradual by going straight into applying neural networks to financial data.

Deep learning with neural networks is all the rage right now so I started messing with it a week ago. Rather than toil on my own I'm going to share my process of trying to get my head around neural networks and finding a model with tradeable signals.

# Prejudices

Before we start I'm going to go on record and say that neural networks probably won't provide any tradeable signals. The reason behind my bias is that security prices are essentially random and it seems as if neural networks are getting the best results in the real world when they are applied to data with "hard edges". Because seucrity prices are essentially randome there aren't really any "hard edges" in finance so intuitively it seems that neural networks will generally meander. Having said this, I hope to be proven wrong.

# LSTM + XLF

I'm going to try and set up a recurrent neural network on XLF price data. The XLF is an ETF that mirrors a basket of financial stocks. We could use the SP500 or an ETF that tracks the SP500 but I thought it would be easier to work an ETF with less components.

For our neural network, we'll be using Long Short-Term Memory ("LSTM"). I approach machine learning models the same way I approach databases, I just want to learn to use them, I don't care how they work. As a result, I'll let [other](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) [sites](https://deeplearning4j.org/lstm) walk you through the wonders of LSTM but for our purposes the important thing you need to know is that they not only allow you to train on features but also allows you to train on the time series of those features as well. This should come in handy when trying to predict our time series data.


# The Toolbox

I'm going to be using [Keras](https://keras.io/) with [Tensor Flow](https://www.tensorflow.org/) as the backend. Keras doesn't support Tensor Flow 1.0 unless you install Keras 2.0 which is a real pain to install so we'll use Keras with Tensor Flow 0.12.  You'll need Python and Postgresql installed on your machine. In the github repo, you'll find a `requirements.txt` with the dependencies you need. If you don't use it already, you should also load virtualenv to sandbox your Python environment.
