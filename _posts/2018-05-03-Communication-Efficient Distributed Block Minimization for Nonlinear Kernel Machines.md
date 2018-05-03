---
layout:     post
title:      非线性核机器的通信高效分布式块最小化
subtitle:   Communication-Efficient Distributed Block Minimization for Nonlinear Kernel Machines
date:       2018-05-03
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Paper
    - 并行计算
    - 非线性核机器
    - 分布式系统
    - SVM
---

## ABSTRACT
Nonlinear kernel machines often yield superior predictive performance on various tasks; however, they suffer from 
severe computational challenges. In this paper, we show how to overcome the important challenge of speeding up kernel 
machines using multiple computers. In particular, we develop a parallel block minimization framework, and demonstrate 
its good scalability in solving nonlinear kernel SVM and logistic regression. Our framework proceeds by dividing the 
problem into smaller subproblems by forming a block-diagonal approximation of the Hessian matrix. The subproblems are 
then solved approximately in parallel. After that, a communication efficient line search procedure is developed to 
ensure sufficient reduction of the objective function value by exploiting the problem structure of kernel machines.
We prove global linear convergence rate of the proposed method with a wide class of subproblem solvers, and our analysis 
covers strongly convex and some non-strongly convex functions. We apply our algorithm to solve large-scale kernel SVM problems 
on distributed systems, and show a significant improvement over existing parallel solvers. As an example, on the covtype dataset 
with half-a-million samples, our algorithm can obtain an approximate solution with 96\% accuracy in 20 seconds using 32 machines, 
while all the other parallel kernel SVM solvers require more than 2000 seconds to achieve a solution with 95\% accuracy. 
Moreover, our algorithm is the first distributed kernel SVM solver that can scale to massive data sets. On the \KDDB dataset 
(20 million samples and 30 million features), our parallel solver can compute the kernel SVM solution within half an hour using 
32 machines with 640 cores in total, while existing solvers can not scale to this dataset.

## 摘要
非线性核机器在各种任务上往往具有优越的预测性能，然而，它们在计算上遇到了挑战。在本文中，我们展示了如何使用多台计算机加速核机器，从而克服挑战。
并且我们开发了一个并行化块最小化框架，在解决非线性核SVM和Logistics回归时证明了其良好的扩展性。我们的框架通过将原问题划分成更小的子问题，
通过形成Hessian矩阵的块对角逼近。然后子问题近似并行求解。除此之外，开发了一种通信高效的线搜索方法，通过利用核机器问题的结构问题来确保
目标函数值充分减少。我们证明了所提出的方法具有广泛的子问题求解器的全局线性收敛速度，并且我们的分析涵盖强凸和一些非强凸函数。
我们应用我们的算法来解决大规模核SVM问题的分布式系统，并显示出明显的改进现有的并行求解器。作为一个例子，在有近50万样本的数据集中，我们的算法可以
在20秒内获得96%准确率的解，而其他并行核SVM求解器需要超过2000秒获得95%准确率的解。此外，我们的算法是第一个分布式核SVM求解器，可以应用于大规模的数据集。

## 关键词
并行计算、非线性核机器、分布式系统、SVM
