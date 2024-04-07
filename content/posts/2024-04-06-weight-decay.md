---
title           : "对线性回归中 Weight Decay 的理解"
date            : 2024-04-06
isCJKLanguage   : true
---

在读《Dive into Deep Learning》关于 Weight Decay 的章节时，我有很多疑问，思索查询后整理成了这篇博文。希望能帮助到有同样疑问的人。

原文在这里：[Weight Decay](https://d2l.ai/chapter_linear-regression/weight-decay.html)

先摆出一个重要的判断：越复杂的 model 越容易 overfit。

一个 model，或者说 function，有几个影响复杂度的属性：
1. features 的数量：即 X 的列数。但这个是 data set 自身的属性，不由 function 决定（当然，我们可以在 function 之外，也就是在数据收集阶段改变它。但这里只聚焦 function）。
2. weights 的取值：其中每个 weight 的大小会决定 model 对相应 feature 的敏感程度 —— 越大的 weight 会让 model 对相应的 feature 更敏感，因为两者的计算是乘法。越小的 weight 会让 model 对相应的 feature 越迟钝。
3. bias 的取值：它的影响极小。
4. 以上三者组成的多项式：主要是次数（degree）和项数的选择。即便仅有一个 feature 也可以组成很多种多项式，比如 $wx^2 + wx + b$，或 $wx^{15} + b$。

到此可以发现，如果从 1 和 3 着手，很难应对 overfit 。于是重点考虑 2 和 4。但 4 很快就被排除了，因为很容易导致变化波动特别大 —— 稍微调整一下次数或多项式的项数，都会让 function 的 output，也就是 y_predict，变化剧烈。

因此，只剩下 3，也就是从 weights 着手。有两个殊途同归的考虑角度：
1. 如前所述，如果 weights 中的部份值很大，那么 model 会对那部份对应的 feature 特别敏感，这种敏感会让 model 更容易 overfit。那么我们就让 weights 的值更均衡，避免一部份值很大，另一部份很小。
2. 越复杂的 model 越容易 overfit。优化的前提是可衡量，衡量 model 复杂度的方法之一是 norm。其中 L2 norm 正好能识别上一条所描述的 weights 值不均衡的现象。

于是，就在 loss 函数里加上对 L2 norm 的惩罚。

总结：缓解 overfitting 的关键是降低 model 的复杂度。影响复杂度的几个关键属性中，最易着手的、最易控制力度的是 weights。这是 Weight Decay 之所以能缓解 overfitting 的大致原理。
