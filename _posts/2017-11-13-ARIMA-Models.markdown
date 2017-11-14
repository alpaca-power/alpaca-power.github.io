---
title:  "ARIMA Models for Time Series Analysis"
date:  2017-11-13 19:54:00
category: [r]
tags: [r]
---

I know there is the `auto.arima` function from the `forecast` library, though we had a homework where we were supposed to
manually find candidate ARIMA models for Unemployment rate prediction by examining its Acf & Pcf graphs. 

Needless to say I had a hard time trying to figure out which Arima models to use. I'm probably wrong because the optimal 
model I found was different from what `auto.arima` gave me, but I'm just going to write down my notes on the process.

Only going to talk about using unemployment rate that were not seasonally adjusted:

```r
library(quantmod)
library(forecast)
library(lubridate)

getSymbols('UNRATENSA', src = 'FRED') # get non-seasonally adjusted unemployment rate

#split into training and testing data
NonSeasonal.Train <- ts(UNRATENSA['/2016'], frequency = 12)
NonSeasonal.Test <- ts(UNRATENSA['2017/'], frequency = 12)

tsdisplay(NonSeasonal.Train) # visualize time series, which is clearly not stationary

# Testing for stationality anyway
adf.test(NonSeasonal.Train) # Nonstationary
kpss.test(NonSeasonal.Train) # Nonstationary

# need to take differential of 1 to remove seasonality
tsdisplay(diff(NonSeasonal.Train, lag = 12, 1), lag.max = 12*5)
# ARIMA(p,0,q)(P,1,Q) P = 1, p ~3, Q = 2, q = ???

tsdisplay(diff(diff(NonSeasonal.Train, lag = 12, 1)), lag.max = 12*5)
# ARIMA(p,1,q)(P,1,Q) P = 4, Q = 1, p = 2, q = ~3?

tsdisplay(diff(diff(NonSeasonal.Train)), lag.max = 12*5)
# ARIMA(p,1,q)(P,0,Q) P ~ 1 (barely), Q = ?, p = 4, q = 4
```

So, based on my guessings (I do not feel confident about the Acf/Pcf graphs), I had 6 candidate models:
* ARIMA(3,0,0)(1,1,2)[12]
* ARIMA(3,0,2)(1,1,2)[12]
* ARIMA(2,1,0)(4,1,1)[12]
* ARIMA(4,1,4)(1,0,2)[12]
* ARIMA(4,1,4)(1,0,3)[12]
* ARIMA(4,1,4)(1,0,3)[12]

None of these model's residuals passed Acf/Pcf tests within 3 lag periods, but  ARIMA(3,0,2)(1,1,2)[12] had the
best results with only 1 Acf and 1 Pcf spikes plus the smallest AICc value and second-smallest RMSE value.

I then tested the models with the Box-Ljung test instead of the simple Box.test (Pierce) taught in class.
Took me a while to figure out what to use for the lags, but per https://robjhyndman.com/hyndsight/ljung-box-test/ 
I used `12*2`. As I didn't learn about it in class I had to search around to learn `fitdf` would take the value
P+Q+p+q for respective models.

Anyhow, only ARIMA(3,0,2)(1,1,2)[12], ARIMA(4,1,4)(1,0,2)[12], ARIMA(4,1,4)(1,0,3)[12] passed the test in having high
p-values which means Ho of lack of model fit is not rejected and the model has better prediction quality. 

Just a side note, `auto.arima` suggested the best model to be ARIMA(4,1,4)(1,0,0)[12] which fails the Box.Ljung test quite badly.

Then we need to backward test the ARIMA models to find the best model for each quarter. Naturally this is necessary
and I also wanted to know the consistency of the model for past data. The assignment happened to ask for backtesting
seven periods of time frames and I had 6 models, definitely super tedious and being lazy I don't want to copy and paste
like an idiot. There's probably a better way to approach the problem, but mine was as follows:

```r
training_sets <- seq(from = ymd('2015-09-01'),
                     to = ymd('2017-09-01'),
                     by = 'quarter')

ns_best_model <- matrix(NA, nrow = 0, ncol = 6, 
                     dimnames = list(NULL, c('set', 
                                             'RMSE', 
                                             'AICC(d0)(D1)',
                                             'AICC(d1)(D1)',
                                             'AICC(d1)(D0)', 
                                             'BIC')))

for(i in 1:(length(training_sets)-1)){
  training_end <- as.character(training_sets[i])
  testing_start <- as.character(training_sets[i+1])
  
  training <- ts(UNRATENSA[paste0('/',training_end)],
                 frequency = 12)
  testing <- ts(UNRATENSA[paste0(training_end,'/',testing_start)], 
                frequency = 12)
  
  M1 <- Arima(training, order = c(3,0,0), seasonal = c(1,1,2))
  M2 <- Arima(training, order = c(3,0,2), seasonal = c(1,1,2))
  M3 <- Arima(training, order = c(2,1,0), seasonal = c(4,1,1))
  M4 <- Arima(training, order = c(4,1,4), seasonal = c(1,0,0))
  M5 <- Arima(training, order = c(4,1,4), seasonal = c(1,0,2))
  M6 <- Arima(training, order = c(4,1,4), seasonal = c(1,0,3))
  
  Models <- c('ARIMA(3,0,0)(1,1,2)[12]',
              'ARIMA(3,0,2)(1,1,2)[12]',
              'ARIMA(2,1,0)(4,1,1)[12]',
              'ARIMA(4,1,4)(1,0,0)[12]',
              'ARIMA(4,1,4)(1,0,2)[12]',
              'ARIMA(4,1,4)(1,0,3)[12]')
  
  RMSE1 <- accuracy(M1,test = testing)[,'RMSE']
  RMSE2 <- accuracy(M2,test = testing)[,'RMSE']
  RMSE3 <- accuracy(M3,test = testing)[,'RMSE']
  RMSE4 <- accuracy(M4,test = testing)[,'RMSE']
  RMSE5 <- accuracy(M5,test = testing)[,'RMSE']
  RMSE6 <- accuracy(M6,test = testing)[,'RMSE']
  
  RMSE <- cbind(RMSE1, RMSE2, RMSE3, RMSE4, RMSE5, RMSE6)
  AICC1 <- cbind(M1$aicc, M2$aicc) # cannot compare AICc for models with different differencing
  AICC2 <- cbind(M4$aicc, M5$aicc, M6$aicc)
  BIC <- cbind(M1$bic, M2$bic, M3$bic, M4$bic, M5$bic, M6$bic)
  AICC <- cbind(M1$aicc, M2$aicc, M3$aicc, M4$aicc, M5$aicc, M6$aicc)

  best <- cbind(quarter(training_end, with_year = T), 
                Models[which.min(RMSE)], 
                Models[which.min(AICC1)],
                Models[3],
                Models[which.min(AICC2)+3],
                Models[which.min(BIC)])
  ns_best_model <- rbind(ns_best_model, best)
}
```

This code took longer than I thought it would take to finish, so it definitely needs some cleaning up or
efficiency work. Conclusion was that ARIMA(3,0,2)(1,1,2)[12] ended up being valued as the best model based on
AICc critereon while ARIMA(2,1,0)(4,1,1)[12] remained the one with smallest RMSE overally. Makes me wonder
if I am doing something wrong but... we'll see.

Then just for the heck of it I tried to see how the model performed for each time frame:
```r
for(i in 1:(length(training_sets)-1)){
  training_end <- as.character(training_sets[i])
  testing_start <- as.character(training_sets[i+1])
  
  training <- ts(UNRATENSA[paste0('/',training_end)],
                 frequency = 12)
  testing <- ts(UNRATENSA[paste0(training_end,'/',testing_start)], 
                frequency = 12)

  fit <- Arima(training, order = c(3,0,2), seasonal = c(1,1,2))
  plot(forecast(fit, h = 6), xlab = quarter(testing_start, with_year = T))
  lines(ts(UNRATENSA[paste0('/', testing_start)], frequency = 12))
  lines(fitted(fit), col = 'red')
}
```
The model seems to work pretty well making quarterly forecasts for the most part (of course there were times when it
forecasted off, but the forecasts were much better for the short term than for, say, one year.


#### Extra:

Of course, I would like to know how including fourier-transformed past data in `xreg` would work, since a lot of tutorials 
I found seemed to use time series as signals because it produces better results:

```r
bestfit <- list(aicc=Inf)
for(i in 1:(12/2)){
  fit <- auto.arima(NonSeasonal.Train, 
                    xreg=fourier(NonSeasonal.Train, K=i), seasonal=T)
  if (fit$aicc < bestfit$aicc) {
    bestfit <- fit
  }
  else break;
}
NonSeasonal.Fcst <- forecast(bestfit, xreg=fourier(NonSeasonal.Train, K=i, h=12*2))
plot(NonSeasonal.Fcst)
lines(ts(UNRATENSA, frequency = 12))
lines(fitted(bestfit), col = 'red')
```
This time `auto.arima` found ARIMA(3,1,1)(2,0,0)[12] as the best model, and forecast-wise it looked much better than what
ARIMA(3,0,2)(1,1,2)[12] gave. However it seemed to have null values and the model could not be tested...


At any rate, shall write down my notes on feature selection in next blog update.
