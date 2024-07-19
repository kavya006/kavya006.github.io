---
title: "Tensorflow tf.data Pipeline for Multiple Time Series"
date: 2024-07-19
categories:
  - tutorials
tags:
  - python
  - tensorflow
  - time series forecasting 
---

One of the official Tensorflow tutorials was for Time Series Forecasting. If you haven’t gone through it yet, I highly recommend going through the tutorial. [Time Series Forecasting](https://www.tensorflow.org/tutorials/structured_data/time_series)

Fast forward a few months, I came across a similar problem at work. I had to work with 10k+ different time series at once and develop models for such a dataset. I had written a class for Preprocessing from scratch which resulted in almost (500 lines of code for just the preprocessing). This preprocessing class could do all the tasks required for this particular task.

As the person I am, I wanted to further optimize this code and also learn to use tf.data pipeline for preprocessing. I remembered how I learnt about the Window Generator. Now comes the issue, Window Generator was for a single time series. I searched online for resources if there was an existing post of extending the WindowGenerator class for multiple time series but with no luck.

Here, I am sharing the WindowGenerator which works with Multiple Time Series

--- 

### WindowsGenerator — Brief Introduction

WindowsGenerator is the preprocessing class from the Tensorflow’s Time Series Tutorial. It implements the following methods:

1. `constructor` — takes a single time series dataframe (`train`, `val` and `test` dfs) and stores the relevant slicing information for inputs and labels.
2. `split_window` — takes a single array of size total_window_sizeand splits it to inputs and labels.
3. `plot` — plots a min(`max_plots`, `batch_size`) number of examples. Shows all inputs, labels and predictions in the example plot.
4. `make_dataset` — converts the input array to a `tf.data.Dataset` and applies the split_window method on the dataset. Returns us the input to the different models.

### Details of the Dataset used for the tutorial

I have used the same weather dataset used in the original tutorial. I am trying to use '`T (degC)` as the label column and '`p (mbar)`' and '`rh (%)`’ as the additional regressors to use as additional variables in the multi variate analysis.

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/tf-regression-config.png" />
</div>

Example of the data used for the updated Window Generator. The data has 3 series with the Series ID = 1, 2 or 3. Each Series is a copy of the original weather dataset and was only created for this tutorial.

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/tf-regression-sample.png" />
</div>

### Updated Window Generator for Multiple Time Series
The differences between Original WindowGenerator and MultiSeriesWindowGenerator constructor

- addition of batch_size as a parameter.
- removal of `train_df`, `val_df`, `test_df` as parameters to the init function.
- added `regressor_columns`, `static_columns` for better management of input features to the model.
- addition of `GROUPBY` as a parameter to identify different series in the input data.

<script src="https://gist.github.com/kavya006/672bd8e0788574d1f1bb4d6f85f07585.js"></script>

Now, we add a method to update the `train`, `test` and `val` dfs to the window generator class. The following code does 3 things

1. Takes the input series (`train`, `val`, `test`) and convert them individually into tensors, in the shape (`n_series` x `n_batch` x `n_timesteps` x `n_features`)
2. I have added the `normalization` step inside update_datasets method. This is completely optional through the flag `norm`
3. I have moved the `column_indices` initialization from the `constructor` to the `update_datasets` to update based on the `train_df` every time.

<script src="https://gist.github.com/kavya006/8d61726170b79f7cffd7b61429317c18.js"></script>

The `split_window` and `plot` methods do not need to be updated and can be used from the original WindowGenerator class.

To keep the interface as same as possible, I have renamed `WindowGenerator.make_dataset` to `MultSeriesWindowGenerator.make_cohort` and add a new `make_dataset` method which will call the make_cohort method internally.

<script src="https://gist.github.com/kavya006/c36c01f0e406d871b8cfd1a36d964bd4.js"></script>

The above make_dataset method does 3 things

1. It will call `make_cohort` for each of the series in data. Note that `make_cohort` returns a `tf.data.Dataset` as shown in the original tutorial.
2. It will then zip all different `tf.data.Dataset` and then we stack all the inputs and labels as shown in the `stack_windows` function to create a single `Dataset` object.
3. We `unbatch` the data, `shuffle` it and then `batch` the data again as per our defined `batch_size`.
4. Finally we `prefetch` the Dataset and return this as the output.

The rest of the code remains same. At this point, I plotted an example to see if everything is working as expected.

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/tf-regression-plot1.png" />
</div>

I’ve tested using the `MultiSeriesWindowGenerator` with the `Baseline` model provided in the tutorial. You can see the results here.

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/tf-regression-plot2.png" />
</div>


--- 

### Final Thoughts
Thank you for following the tutorial so far. I hope I helped at least one person with their quest on finding a WindowGenerator which works for multiple time series. You can find the full code in the following git link: [WindowGenerator_with_Multiple_Time_Series.ipynb](https://github.com/kavya006/medium_posts/blob/main/WindowGenerator_with_Multiple_Time_Series.ipynb)

Are there any issues in this code? Are there any other features you would want to see implemented? Do let me know in the comments. Thanks in advance.

Happy Reading! and Have a great day everyone :)