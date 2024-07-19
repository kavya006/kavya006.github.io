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

Now the regression task got converted to a classification task. At this moment, the code only supports a single Target label. I haven’t worked out using the code for multi output classification. Currently, this post works only for the multi class classification problem.

### Data Preprocessing 
When it comes to the constructor of the `MultiSeriesWindowGenerator` class, it remains untouched.

Next change is in the preprocessing code of `MultiSeriesWindowGenerator` class.

<script src="https://gist.github.com/kavya006/e5d1029b5b291dcb93daff021ad1eae0.js"></script>

For the regression task, we do not need to separate out the features and targets from the dataset. But the classification requires us to do that.

1. We separate the label columns from the regressor columns and static columns. The reason to do this change becomes clear in the next step.
2. We then reshape the features and targets to have the correct number of dimensions if we do not have enough dimensions in them. For a time series dataset, we need 3 dimensions (batch_size x time steps x n_features)
3. Finally, we return the features and targets separately.
Now, we need to update the update_datasets method to reflect the changes needed to process the data for classification tasks.

<script src="https://gist.github.com/kavya006/a4bec756191d3e02675df089adcca466.js"></script>

1. First, we retrieve features and targets separately from the preprocess_dataset method. Previously, we obtained the entire data together.
2. We only normalize the features and not the targets.
3. After obtaining the normalized features, we concatenate them with the targets in the original order to maintain backward compatibility.

### Plotting Code 
This change is optional and not necessary for the Preprocessor to work with different TensorFlow models. You can skip this part if desired.

<script src="https://gist.github.com/kavya006/d659a95ad14843ba7cfd4948fad07f16.js"></script>

An Example plot from this method is

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/plot1-tf-pipeline-for-multiple-series.png" />
</div>

<div class="container">
<img src="https://kavya006.github.io/assets/images/posts/plot2-tf-pipeline-for-multiple-series.png" />
</div>

Some notes regarding this method:

1. The `plot_col` and `label_col` are now separated. `plot_col` is plotted as a line chart, while `label_col` is plotted as a scatter plot with * markers. The predictions are plotted as a scatter plot with + markers. If the colors match, it indicates a correct classification; if the colors don’t match, it means a classification mistake
2. The actual position of the labels and predictions do not have any correlation with the input features. They are just added to show the difference between labels and predictions.
3. The model predictions are now a one-hot vector of the predictions. So to convert them to class labels, we take an argmax. This step was not needed in the regression task.

### Finally the Baseline model 

<script src="https://gist.github.com/kavya006/1343750fadb525c94a97ac046ef2f1ad.js"></script>

Finally, we just update the loss function from MSE to SparseCategoricalCrossEntropy to work with the one hot vector which our model outputs.

--- 

That’s it, folks. That concludes the necessary changes to make the MultiSeriesWindowGenerator work for classification tasks. You can find the full code available at the following link: — [Window Generator for Classification](https://github.com/kavya006/medium_posts/blob/main/WindowGenerator_with_Multiple_Time_Series_Seq_2_Seq_Classification.ipynb)

I hope this article proves helpful to anyone seeking assistance with this topic. Thank you for taking the time to read through it. If you encounter any issues with the code, please let me know. If you have any requests or new features you’d like to see implemented, please share them in the comments, and I’ll be happy to try to accommodate them.

Wishing you a great and productive day! And Happy Coding :)