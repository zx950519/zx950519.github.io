---
layout:     post
title:      通信高效的稀疏回归
subtitle:   Communication-efficient Sparse Regression
date:       2018-05-02
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Paper
    - ML
    - JMLR
    - 回归
---

>不积跬步 无以至千里

## ABSTRACT  
We devise a communication-efficient approach to distributed sparse regression in the high-dimensional setting. 
The key idea is to average debiased or desparsified lasso estimators. We show the approach converges at the same 
rate as the lasso as long as the dataset is not split across too many machines, and consistently estimates the 
support under weaker conditions than the lasso. On the computational side, we propose a new parallel and 
computationally-efficient algorithm to compute the approximate inverse covariance required in the debiasing approach, 
when the dataset is split across samples. We further extend the approach to generalized linear models.  
## 摘要  
我们设计了一种高效的通信方法来实现高维设定下的分布式稀疏回归。核心思路是平均惩罚或低密度化lasso估计量。只要数据集不是跨多台机器进行分割，
该方法就会以和Lasso一致的速率收敛，并且始终在比Lasso更弱的条件下支持估计。在计算上，我们提出了一种新的并行和计算效率高的算法，用于在去偏
方法中所需的近似逆协方差，即数据集在样本间分离。我们进一步将该方法扩展到广义线性模型。
## 关键词
稀疏回归、Lasso
