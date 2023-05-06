---
title           : "3Blue1Brown《线性代数的本质》学习笔记"
date            : 2023-05-06
isCJKLanguage   : true
---

还没有学完，这条笔记会持续更新。

B 站视频：[线性代数的本质](https://www.bilibili.com/video/BV1ys411472E)。

## 一、Matrix Multiplication（矩阵乘法）

几何意义：线性变换（linear transformation）。

例如 $\vec{v}$ = $A\vec{x}$ 表示把向量 $\vec{x}$ 经过一个矩阵 $A$ 代表的线性变换后得到向量 $\vec{v}$。举例如下：

### 关于向量 $\vec{x}$

有一个二维向量 $\vec{x} = \begin{bmatrix}2\\\\1\end{bmatrix}$，它的原点当然是座标原点，端点是横轴值为 2，纵轴值为 1 的点。

我们把这个二维空间的基向量定义为：
* 横轴基向量 $i = \begin{bmatrix}1\\\\0\end{bmatrix}$。(横轴上的一个与原点距离是 1 的点)
* 纵轴基向量 $j = \begin{bmatrix}0\\\\1\end{bmatrix}$。(纵轴上的一个与原点距离是 1 的点)

注：2 维空间的轴既可以被称为 $x$ 轴和 $y$ 轴，也可以称为 $i$ 轴和 $j$ 轴。这里选择的称呼方法是后者，但和前者意思一样，只是叫法不同。

如果用基向量来表示 $\vec{x}$，就是：$\vec{x} = 2 * i + 1 * j$ 。

### 关于矩阵 $A$

有一个矩阵 $A = \begin{bmatrix}3 & 0 \\\\ 0 & 4\end{bmatrix}$，表示一个线性变换（把横轴拉长成原始的 3 倍，把纵轴拉长成原始的 4 倍）。

那么经过 $A$ 代表的线性变换后，新的基向量 $\hat{i}$ 和 $\hat{j}$ （注意带了帽子）在原空间里的位置变成了：
* $\hat{i} = \begin{bmatrix} 3 \\\\ 0 \end{bmatrix}$
* $\hat{j} = \begin{bmatrix} 0 \\\\ 4 \end{bmatrix}$

### 关于结果向量 $\vec{v}$

$\vec{v} = 2 * \hat{i} + 1 * \hat{j}= 2 * \begin{bmatrix}3\\\\0\end{bmatrix} + 1 * \begin{bmatrix}0\\\\4\end{bmatrix} = \begin{bmatrix}6\\\\4\end{bmatrix}$

## 二、Column Space（（矩阵的）列空间）

几何意义：

矩阵 $A$ 的列空间是指任一向量 $\vec{x}$ 经过 $A$ 线性变换之后的结果向量的集合。换句话说：矩阵 $A$ 的列空间是 $A\vec{x}$ 产生的所有向量张成（span）的空间。

## 三、Determinant（行列式）

几何意义：
* 用来测量线性变换对空间的伸缩程度。比如：在 2 维空间里测量面积变化，在 3 维空间里测量体积变化。
* Determinant = 0 表示线性变换会降维。比如 $\vec{x}$ 是一个 3 维向量（张量）, $\vec{v} = A\vec{x}$ 是一个 2 维向量或 1 维点。此时 $\vec{x}$ 没有逆矩阵，即 $\vec{x}^{-1}$ 不存在。

举例（2 维空间）：

有一个向量 $\vec{x} = \begin{bmatrix}1\\\\1\end{bmatrix}$，它的端点、座标原点和横纵两轴合围而成的矩形的面积是 1。

有一个矩阵 $A = \begin{bmatrix}3 & 0 \\\\ 0 & 4\end{bmatrix}$ 代表线性变换。

线性变换之后的结果向量 $\vec{v} = A\vec{x} = \begin{bmatrix}3 & 0 \\\\ 0 & 4\end{bmatrix} \odot \begin{bmatrix}1\\\\1\end{bmatrix} = \begin{bmatrix}3 \\\\ 4\end{bmatrix}$。它围起来的矩形的面积是 12。

那么 determinant($A$) = $12 \div 1 = 12$。表示经过线性变换 $A$ 之后，空间大小是原空间的 12 倍。


## 四、Rank（秩）

代表经过线性变换之后空间（列空间）的维数。用来形容线性变换的维数变化。举例来说：

* 当 Determinant($A$) = 0。如果一个向量经过 $A$ 线性变换后，维数变为 2，那么我们就称矩阵 $A$ 的秩是 2。

* 如果线性变换前后的秩相同，也就是维数相同，则称 $A$ 是满秩的。而非满秩则代表线性变换 $A$ 会降维。
