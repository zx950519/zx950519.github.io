---
layout:     post
title:      支持向量机超参数优化的重采样过程的经验评估
subtitle:   Empirical Evaluation of Resampling Procedures for Optimising SVM Hyperparameters
date:       2018-05-02
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - ML
    - JMLR
    - SVM
    - 超参数
    - 优化
---

>不积跬步 无以至千里

## ABSTRCT
Tuning the regularisation and kernel hyperparameters is a vital step in optimising the generalisation performance of kernel
methods, such as the support vector machine (SVM). This is most often performed by minimising a resampling/cross-validation 
based model selection criterion, however there seems little practical guidance on the most suitable form of resampling. 
This paper presents the results of an extensive empirical evaluation of resampling procedures for SVM hyperparameter selection, 
designed to address this gap in the machine learning literature. We tested 15 different resampling procedures on 121 binary 
classification data sets in order to select the best SVM hyperparameters. We used three very different statistical procedures 
to analyse the results: the standard multi- classifier/multi-data set procedure proposed by Dem\v{s}ar, the confidence intervals 
on the excess loss of each procedure in relation to 5-fold cross validation, and the Bayes factor analysis proposed by Barber. 
We conclude that a 2-fold procedure is appropriate to select the hyperparameters of an SVM for data sets for 1000 or more datapoints, 
while a 3-fold procedure is appropriate for smaller data sets.

## 摘要
优化正则化和核超参数是优化核方法的推广性能的关键步骤，如支持向量机（SVM）。这通常是通过最小化重采样/交叉验证的模型选择准则来执行的，然而，
在最合适的重采样形式上似乎没有什么实用的指导。本文介绍了重采样超参数选择的钢钒评价结果，并为机器学习领域解决了这一难题。我们测试了15种不
同的采样程序121二分类数据集，以选择最佳的SVM参数。我们用了三个不同的统计方法来分析结果：标准的多分类器/多数据集的程序由Dem\v{s}ar提出，
在超额损失的每个步骤(?程序)的置信区间与5折交叉检验有关，以及Barber提出的贝叶斯因子分析。我们的结论是，2折(?程序)在1000个或者更多的数据点
条件下比较适合选出支持向量机的参数，3折(?程序更适合较小的数据集)。

## 关键词  
ML、JMLR、SVM、超参数、优化
