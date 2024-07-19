---
title: "Tensorflow tf.data Pipeline for multiple Time Series Sequence to Sequence Classification"
date: 2024-07-19
categories:
  - tutorials
tags:
  - python
  - tensorflow
  - time series forecasting 
---

Time series data analysis plays a crucial role in various domains, ranging from finance and healthcare to weather forecasting and industrial processes. In a previous article, we explored the implementation of the MultiSeriesWindowGenerator class in TensorFlow, which proved useful for time series forecasting tasks.

In the comments, one reader requested to see how the MultiSeriesWindowsGenerator defined in the said post could be modified to work for classification tasks instead of the typical regression tasks. I would like to thank Simone Alessia for the suggestion. In this post, I will attempt to adapt it for classification. For those who want to refer to my previous post, you can find it here - [Tensorflow Data Pipeline for Multiple Time Series | Medium](https://)

One thing to note before I start off - I will only mention the changes I have made. If a change is not mentioned, it means the code remains the same as in the last post.

With all the disclaimers out of the way, let's get started
--- 

Now, let's dive into the list of changes made to the code to support classification:
1. Data Generation
2. Dataset Preprocessing (preprocess_dataset, update_datasets)
3. Plotting function (plot)
4. Model (call method of the Baseline model)

### Data Generation 
This is an obvious change since the data requirements differ between time series forecasting and classification tasks. To get a dataset suitable for the classification task, I took the LABEL column 'T (degC)' and divided it into 4 classes based on the quantiles.

<script src="https://gist.github.com/kavya006/430ea91ca61ded27a98059122568248c.js"></script>