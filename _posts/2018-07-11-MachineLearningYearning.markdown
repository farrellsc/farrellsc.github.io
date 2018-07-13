---
layout:     post
title:      "Machine Learning Yearning"
subtitle:   "Andrew Ng's new book"
date:       2018-07-11 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - Leetcode
    - Andrew Ng
    - AI
---

### 0. Brief

![](/img/in-post/2018-07-11-MachineLearningYearning/Cover.png)

"Machine Learning Yearning" is a handy brochure written by Andrew Ng that tells useful tricks and things to notice when coders are structuring cool ML projects. This book is still an ongoing draft, to know more about it you may visit [here](http://www.mlyearning.org/).

**Why is ML taking off ?** Key progress: Data Availability & Computational Scale. So one of the more reliable ways to improve an algorithm’s performance today is still to (i) train a bigger network and (ii) get more data.

### 1. Setting up development and test sets
**Definitions**:
- **Training set​**: Which you run your learning algorithm on.
- **Dev (development) set​**: Which you use to tune parameters, select features, and make other decisions regarding the learning algorithm. Sometimes also called the hold-out cross validation set​.
- **Test set​**: which you use to evaluate the performance of the algorithm, but not to make any decisions regarding what learning algorithm or parameters to use.

**Facts you should know about dev set**:
- Choose dev and test sets to reflect data you expect to get in the future and want to do well on.
- The size of dev/test set is correlated to the granularity of the accuracy. Too big a dev set is meaningless while too small a dev set is not informative enough. A 70/30 split is for medium-sized datasets, for larger datasets dev set doesn't need such a big split.
- Dev set and Test set should be of the same distribution. This narrows the options when things go wrong to overfitting dev set, in which case you should get more dev set data.

### 2. Evaluation metrics

\-|True|False
:--:|:--:|:--:
Positive|TP|FP
Negative|TN|FN

- **TP**: answer is positive, label is true
- **FP**: answer is positive, label is false
- **TN**: answer is negative, label is true
- **FN**: answer is negative, label is false
  
**Metrics**:
- **Accuracy**: $\frac{TP+FN}{TP+FP+TN+FN}$
- **Precision**: $\frac{TP}{TP+FP}$
- **Recall**: $\frac{TP}{TP+FN}$
- **F1**: $\frac{2Prec\*Recall}{Prec+Recall}$
- **Runtime**

**Optimizing multiple(N) metrics**: set N-1 metrics to thresholds, marked as "satisfactory", then try optimizing one most important metric.

**When you need to change your dev set / metrics**:
1. The actual distribution you need to do well on is different from the dev/test sets.
2. You have overfit to the dev set
3. The metric is measuring something other than what the project needs to optimize.

### 3. Experiment Loops
![](/img/in-post/2018-07-11-MachineLearningYearning/Experiment-Loop.png)
**The faster you can implement this loop, the faster you will make the progress**:
1. Start off with some idea​ on how to build the system.
2. Implement the idea in code​.
3. Carry out an experiment​ which tells me how well the idea worked. (Usually my first few ideas don’t work!) Based on these learnings, go back to generate more ideas, and keep on iterating.

### 4. Basic Error Analysis
**Things worth noticing**:
1. Look at misclassified examples in dev set to start with. You can analysis multiple misclassification problems in parallel. Use a excel sheet to find the major problems in the model.
2. It's not uncommon to tolerating mislabeled examples to start with, but if the portion of mislabeling accounts for a major portion of misclassification (such as 0.6% mislabeling in 2% error).
3. When you have a large dev set, split it into two sets: eyeball set and blackbox set. Eyeball set (containing 20-100 mistakes) is for misclassification observation while blackbox set (size same as dev for granularity reasons) is for parameter tuning after you have extracted fixure patterns from eyeball set. This measurement would offset overfitting from eyeball set.