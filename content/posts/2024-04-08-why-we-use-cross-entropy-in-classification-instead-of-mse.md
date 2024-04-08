---
title           : "为什么 classification 要用 cross entropy 来表达 loss，而不像 regression 那样用 MSE？"
date            : 2024-04-08
isCJKLanguage   : true
---

在读《Dive into Deep Learning》关于 regression 的章节时，我产生了这个疑问，思索查询后整理成了这篇博文。

原文在这里：[Loss Function](https://d2l.ai/chapter_linear-classification/softmax-regression.html#loss-function)

原因之一：

classification 的 predicted label 和 real label 都是概率分布，误差也用概率表达最为自然。我们期望 predicted label 更“极端”、更接近 real label 的 one hot（它就很极端——只有一个值是 1，其余都是 0），数值平均和这个期望背道而驰。

原因之二：

Cross entropy 和 softmax 更般配、节省计算复杂度。前者是对数计算，后者是指数计算。简直天作之合。

原因之三：

用 MSE 的计算方法会“稀释” loss，降低梯度下降的速度。以一个 binary classification 为例：
- True label 是 [1, 0]
- Predicted label 是 [0.01, 0.99]。

可以很直观的看出这个预测错得离谱。我们分别用 MSE 和 Cross entropy 来算一算。

- MSE = $((1-0.01)^2 + (0-0.99)^2)/2$ = 0.49005
- Cross Entropy loss = $-(1*\log 0.01 + 0*\log 0.99)$ = 2.0

从结果可以看出，Cross entropy loss 要比 MSE 大得多，这也就让梯度下降更快，模型优化速度更快。
