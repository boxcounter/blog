---
title           : "样本配比对 Precision 和 Recall 等数据指标的影响与换算"
date            : 2024-05-31
isCJKLanguage   : true
---

## 一、摘要

此文主要讨论了衡量二元分类算法效果的常见数据指标中，
1. 哪些指标会随样本配比改变而变化，哪些不会。样本配比是指测试样本中阴阳两性样本的比例。
2. 会随样本配比变化的这些指标中，如何根据在一种样本配比下的值，换算出在其它不同样本配比下的值。
3. 区分它们并掌握换算方法在业务场景中的意义。

此文的写作动机是在一次内部评审讨论中，导师点出的“样本平衡”话题以及指导激发了我掌握其原理细节的好奇心。在之后的思考与推导过程中，我的同事国杰与我的探讨以及对我推导过程的指正对写作此文帮助良多。

## 二、不同的样本配比对数据指标的影响

常见数据指标有：
1. $Recall$
2. $Precision$
4. $Accuracy$
5. $F\ score$
6. $TPR: True\ Positive\ Rate$
7. $FPR: False\ Positive\ Rate$

如果改变样本配比，上述指标中的 1、5、6 不会改变，而 2、3、4 会改变。我们来分别看看这是因为什么。

### 不受样本配比影响的三个指标

$$
Recall = \frac{TP}{TP + FN}
$$
其中 $TP$ 和 $FN$ 都是阳性样本。所以，不论阴性样本有多少、阴阳两类样本之间的配比多少，都不影响 $Recall$ 的值。

$$
TPR = \frac{TP}{TP + FN}
$$
看公式可以发现它和 $Recall$ 是一样的取值，只是因为在特定使用场景里的需要而有了不同的名字。和 $Recall$ 一样，它也不受样本配比的影响。

$$
FPR = \frac{FP}{FP + TN}
$$
其中 $FP$ 和 $TN$ 都是阴性样本。因为只涉及阴性样本，所以 $FPR$ 也不受样本配比的影响。

### 受样本配比影响的三个指标

和上述三个指标不同，下面三个指标的计算都同时涉及了阳性和阴性这两类样本，所以它们会随着样本配比的改变而变化。

$$
Presicion = \frac{TP}{TP + FP}
$$
其中 $TP$ 是阳性样本数量，$FP$ 是阴性样本数量。

$$
Accuracy = \frac{TP + TN}{TP + FP + TN + FN}
$$
其中 $TP$ 和 $FN$ 是阳性样本数量，$FP$ 和 $TN$ 是阴性样本数量。

$$
F1\ score = 2 \cdot \frac{Precision \cdot Recall}{Precision + Recall} = 2 \cdot \frac{TP}{TP + FP + FN}
$$其中 $TP$ 和 $FN$ 是阳性样本数量，$FP$ 是阴性样本数量。

### 两句话小结

1. 只有同时涉及阴阳两类样本的指标会受到样本配比的影响，也就是会随着样本配比的改变而变化。
2. 只涉及阴阳其中一类样本的指标不受样本配比的影响。

## 三、不同的样本配比下，数据指标的换算方法

这里以换算 $Precision$ 举例。回顾前述 $Precision$ 的定义公式，我们需要两个变量参与计算：$TP$ 和 $FP$。我们可以这样获得它们：
$$
\begin{matrix}
Recall = \frac{TP}{TP+FN} \implies TP = Recall \cdot (TP + FN)\\
FPR = \frac{FP}{FP+TN} \implies FP = FPR \cdot (FP + TN) \\
\end{matrix}
$$

把它们带入 $Precision$ 的定义公式：
$$
Presicion = \frac{TP}{TP+FP} = \frac{Recall \cdot (TP + FN)}{Recall \cdot (TP + FN) + FPR \cdot (FP + TN)}
$$
而其中 $(TP + FN)$ 就是阳性样本的总数量，$(FP + TN)$ 就是阴性样本的总数量。
如前所述，$Recall$ 和 $FPR$ 的值是已知的，并且不随样本配比的改变而变化。

也就是说它们的值都是已知的。因此把它们都带入公式，就能计算出新样本配比下的 $Precision$。

举个例子：
1. 我们根据过往的测试得到了 $Recall = 0.8$ 和 $FPR = 0.2$。
2. 我们希望计算出 9:1 和 99:1 两种样本配比时的 $Precision$。

我们把上述值带入公式会得到：
$$
\begin{matrix}
Precision(9:1) = \frac{0.8 \cdot 9}{0.8 \cdot 9 + 0.2 \cdot 1} = 0.973 = 97.3\% \\
Precision(99:1) = \frac{0.8 \cdot 99}{0.8 \cdot 99 + 0.2 \cdot 1} = 0.997 = 99.7\% \\
\end{matrix}
$$

## 四、换算在业务场景里的意义

在“用 AI 做内容审核”这个场景里，$Recall$ 和 $FPR$ 取决于 AI 模型自身能力和我们使用方法（如 Prompt 和 Temperature）：
1. $Recall$ 的含义是把一批违规内容交给 AI 审核，它能将其中多少个正确地判定为违规。
2. $FPR$ 的含义是把一批合规内容交给 AI 审核，它会将其中多少个错误地判定为违规。

前述换算的意义是：在技术可行性验证的阶段，我们无需按照实际业务环境中的违规内容占比来收集样本，这就降低了收集样本的复杂度。

举个例子：假设我们业务里违规内容占比是 0.1%。如果按照这个占比来收集样本，会遇到这样的两难：

一、我们收集 10,000 个样本用来测试。其中包含 10 个违规样本和 9,990 个合规样本。但是因为违规样本太少了，测试结果不稳定，容易波动。

二、为了解决上述不稳定问题，我们需要增加违规样本。比如我们要把违规样本增加到 100 个，那么我们还需同时把合规样本增加到 99,990。这样才能维持 0.1% 的违规内容占比。

但有了换算之后，我们可以简化准备样本的复杂度，比如我们可以按 1:1 的配比收集 1,000 个样本。然后测试 AI 的能力并记录 $Recall$ 和 $FPR$ 这两个不受样本配比影响的指标。最后根据实际业务环境中违规内容的占比来换算出在实际业务环境中的其他指标比如 $Precision$。