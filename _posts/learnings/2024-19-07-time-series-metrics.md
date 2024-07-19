---
title: "Metrics for Time Series Forecasting"
date: 2019-04-18T15:34:30-04:00
categories:
  - learnings
tags:
  - metrics
  - time series forecasting
---

## Metrics for Time Series Forecasting

In this story, I would like to go through different metrics I have personally used to evaluate the predictions of the models.
- Mean Squared Error
- Mean Absolute Error
- Mean Absolute Percentage Error
- Symmetric Mean Absolute Percentage Error

Evaluating the performance of time series forecasting models is crucial for understanding their accuracy and effectiveness. In this article, we will critically examine these metrics, considering their strengths, limitations, and suitability for different scenarios

### Mean Squared Error
```python
mse = sum((y_true - y_pred)**2)
```
Used when we care more about the larger errors more than the smaller errors. MSE is very high when the predictions are way off the actuals, i.e., we penalize larger deviations more than the smaller ones. There is the issue of the units of the error not matching the units of our predictions. 

To mitigate this, we generally take the square root of the MSE to get the Root Mean Squared Error (RMSE). While MSE and RMSE provide a measure of forecasting accuracy, they lack interpretability and can be challenging to explain to stakeholders.

### Mean Absolute Error
```python
import numpy as np 
mae = np.average(y_pred - y_true)
```
MAE is probably the most explainable error metric which can be used to understand how good / bad we are doing with our forecasts. It has the same units as that of the actuals and thus is very easy to explain. MAE of 1 unit implies that on an average, we deviate by 1 unit in our forecasts.

However, it does not consider the relative scale of the actual data, making it difficult to differentiate between series with different scales. For example, a deviation of 3 units in a series with actuals ranging from 5 +/- 3 is not directly comparable to a deviation of 6 units in a series with actuals ranging from 40 +/- 10. When comparing forecast errors across series with different scales, it may be necessary to normalize or transform the data to ensure fair comparisons.

### Mean Absolute Percentage Error
```python
import numpy as np 
mape = np.average(1 - (y_pred/y_true))
     = np.average(abs(y_true - y_pred)/y_true)
```
MAPE tries to incorporate the relative scale of the actuals which is lacking in MAE. MAPE tells us how much the percentage change is in the actual vs the predicted. Let's say, a MAPE of 10% indicates that we are deviating by 10% on the actuals i.e., if the actual was 10, then we might have forecasted either 9 or 11 units. MAPE can be used to compare different series since it is not dependent on the scale of the data.

However, one major drawback of MAPE is that it becomes undefined when the actual values are zero, resulting in division by zero (Denominator = 0 => MAPE = infinity). This limitation restricts its use in scenarios where zero values are present in the time series.

### Symmetric Mean Absolute Percentage Error (sMAPE)
```python
import numpy as np 
smape = np.average(1 - (y_pred/((y_true + y_pred)/2))) 
    = np.average(1 - (2 * y_pred / (y_true + y_pred)))
    = np.average(((y_true + y_pred) - 2 * y_pred)/(y_true + y_pred)) 
    = np.average((y_true - y_pred) / (y_true + y_pred)) 
```
To avoid the problem where MAPE is not defined when the actual = 0, we take the mean of actual and predicted as the denominator instead of just the actual as in the case with MAPE. This means that for sMAPE to be undefined, both the actual and predicted should be 0.

If MAPE is measures the deviation in actual and predicted as compared to the actual value, sMAPE compares the deviation in actual and predicted as compared to the mid point of the actual and predicted value. MAPE focuses on the absolute percentage deviation from the actual values, while sMAPE considers the symmetry of the deviation relative to the midpoint between predicted and actual values.

Similar to MAPE, sMAPE does not depend on the scale of the data we are operating on, and thus can be used for comparing the errors of different series.

One huge drawback of using sMAPE is the explainability. sMAPE is simply not explainable. Also, sMAPE biases towards over-forecasting and penalizes the under-forecasting. This comes from the denominator of the formula, where y_true + y_pred is lower when we over-forecast (y_true < y_pred) as opposed to the under-forecast (y_true > y_pred) .

---

When evaluating time series forecasting models, it is crucial to consider the strengths and limitations of different metrics. 
- While MSE and RMSE emphasize larger errors, they lack interpretability. 
- MAE provides a more intuitive understanding but fails to account for the relative scale of the data. 
- MAPE considers the percentage deviation but becomes undefined when actual values are zero. 
- sMAPE addresses the issue of undefined values but sacrifices explainability and introduces a bias towards over-forecasting. 

Choosing the appropriate metric depends on the specific objectives, stakeholders, and characteristics of the time series data being analyzed.