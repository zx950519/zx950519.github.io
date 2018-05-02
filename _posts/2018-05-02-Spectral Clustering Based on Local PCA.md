---
layout:     post
title:      基于局部PCA的谱聚类
subtitle:   Spectral Clustering Based on Local PCA
date:       2018-05-02
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Paper
    - PCA
    - 谱聚类
    - ML
---

>不积跬步 无以至千里

## ABSTRACT
We propose a spectral clustering method based on local principal components analysis (PCA). After performing local PCA 
in selected neighborhoods, the algorithm builds a nearest neighbor graph weighted according to a discrepancy between 
the principal subspaces in the neighborhoods, and then applies spectral clustering. As opposed to standard spectral 
methods based solely on pairwise distances between points, our algorithm is able to resolve intersections. We establish 
theoretical guarantees for simpler variants within a prototypical mathematical framework for multi-manifold clustering, 
and evaluate our algorithm on various simulated data sets.

## 摘要
我们提出了一种基于局部主成分分析的的谱聚类方法。在选定邻居进行主成分分析后，该算法根据主要子空间之间的差异建立最近邻居加权图，然后应用谱聚类。
相对于标准谱聚类方法考虑在点对间距离，我们的算法能解决交叉点。我们建立的理论为简化数学框架中的变种多流型聚类提供了支持，并且叜各种模拟数据集
上评估了我们的算法。

## 关键词
PCA、谱聚类、聚类、ML
