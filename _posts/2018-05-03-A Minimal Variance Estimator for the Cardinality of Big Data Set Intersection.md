---
layout:     post
title:      大数据集交叉口基数的最小方差估计
subtitle:   Paper
date:       2018-05-03
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Paper
    - 流式计算
    - 最大似然方法
    - 最小化方差
---

## ABSTRACT
In recent years there has been a growing interest in developing "streaming algorithms" for efficient processing and querying 
of continuous data streams. These algorithms seek to provide accurate results while minimizing the required storage and the 
processing time, at the price of a small inaccuracy in their output. A fundamental query of interest is the intersection 
size of two big data streams. This problem arises in many different application areas, such as network monitoring, database systems, 
data integration and information retrieval. In this paper we develop a new algorithm for this problem, based on the 
Maximum Likelihood (ML) method. We show that this algorithm outperforms all known schemes in terms of the estimation’s quality 
(lower variance) and that it asymptotically achieves the optimal variance.

## 摘要
近年来，人们越来越感兴趣的是开发用于高效处理和查询连续数据流的流式算法。这些算法试图提供精确的结果，同时以最小的输出不准确的代价最小化所需的
存储和处理时间。兴趣的基本查询是两个大数据流的相交大小。这个问题出现在许多不同的应用领域，如网络监控、数据库系统、数据集成和信息检索。
在本文中，我们开发了一种新的算法，基于最大似然（ML）方法解决这个问题。我们表明，就质量（较低的方差）而言该算法优于所有已知的方案，
并且它渐进地达到最佳方差。

## 关键词
流式计算、最大似然方法、最小化方差
