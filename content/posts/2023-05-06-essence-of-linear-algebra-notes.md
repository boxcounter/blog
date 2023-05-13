---
title           : "《线性代数的本质》学习笔记"
date            : 2023-05-13
isCJKLanguage   : true
---

声明：不保证正确。仅作为我学习过程中的思考记录，供日后温习之用。

B 站视频：[线性代数的本质（3Blue1Brown）](https://www.bilibili.com/video/BV1ys411472E)

## Basis Vector（基向量）
一个空间的基础构成元素，该空间中的任意元素都可以用基向量来表达。倒过来说，空间的基向量张成（span）了整个空间。

举例：  
2 维空间（面）中有 2 个基向量：
* 水平方向上的基向量 $i = \begin{bmatrix}1\\\\\\\\0\end{bmatrix}$（终点是横轴上的一个与原点距离为 1 的点）
* 垂直方向上的基向量 $j = \begin{bmatrix}0\\\\\\\\1\end{bmatrix}$（终点是纵轴上的一个与原点距离为 1 的点）  

此 2 维面中的向量 $\vec{x} = \begin{bmatrix} 2 \\\\\\\\ 3 \end{bmatrix}$ 用基向量表达就是 $\vec{x} = 2i + 3j$。

举例：  
3 维空间中有 3 个基向量：
* $i = \begin{bmatrix} 1 \\\\\\\\ 0 \\\\\\\\ 0 \end{bmatrix}$
* $j = \begin{bmatrix} 0 \\\\\\\\ 1 \\\\\\\\ 0 \end{bmatrix}$
* $k = \begin{bmatrix} 0 \\\\\\\\ 0 \\\\\\\\ 1 \end{bmatrix}$

此 3 维空间中有一个向量 $\vec{x} = \begin{bmatrix} 2 \\\\\\\\ 3 \\\\\\\\ 4 \end{bmatrix}$ 用基向量表达就是 $\vec{x} = 2i + 3j + 4k$。

## Linear Transformation（线性变换）
几何意义：
不太好用文字描述，建议看视频中的《03 - 矩阵与线性变换》。简单点说，线性变换的本质是产生新的基向量，并用新的基向量来描述线性变换后的向量。

举例（2 维面）：
* $i = \begin{bmatrix} 1 \\\\\\\\ 0 \end{bmatrix}$
* $j = \begin{bmatrix} 0 \\\\\\\\ 1 \end{bmatrix}$

此 2 维面中有一个向量 $\vec{x} = \begin{bmatrix} 2 \\\\\\\\ 3 \end{bmatrix}$，用基向量表达就是 $\vec{x} = 2i + 3j$。

如果我们进行一个逆时针 90º 的旋转（即线性变换），那么新基向量会是：
* $\hat{i} = \begin{bmatrix} 0 \\\\\\\\ 1 \end{bmatrix}$
* $\hat{j} = \begin{bmatrix} -1 \\\\\\\\ 0 \end{bmatrix}$

我们把向量 $\vec{x}$ 经过这个旋转后得到的新向量称为 $\vec{v}$。用新的基向量来表达就是：$\vec{v} = 2\hat{i} + 3\hat{j}$。它的坐标是$\begin{bmatrix} -3 \\\\\\\\ 2\end{bmatrix}$。

总结一下就是，线性变换前后的基向量不同，但向量用基向量描述的表达式是不变的：
* 线性变换之前的向量 $\vec{x}$ 用原基向量 $i$ 和 $j$ 描述： $\vec{x} = 2i + 3j$
* 线性变换之后的向量 $\vec{v}$ 用新基向量 $\hat{i}$ 和 $\hat{j}$ 描述： $\vec{v} = 2\hat{i} + 3\hat{j}$。

## Matrix Multiplication（矩阵乘法）
几何意义：
与一个矩阵相乘意味着做一次线性变换。矩阵的每个列都是变换后的一个基向量。

仍然以横纵坐标系的 2 维面举例，两个基向量分别是：
* 水平方向上的基向量 $i = \begin{bmatrix}1\\\\\\\\0\end{bmatrix}$
* 垂直方向上的基向量 $j = \begin{bmatrix}0\\\\\\\\1\end{bmatrix}$

此 2 维面中有一个向量 $\vec{x} = \begin{bmatrix} 2 \\\\\\\\ 3 \end{bmatrix}$，用基向量表达就是 $\vec{x} = 2i + 3j$。

### 方阵
有一个矩阵 $A = \begin{bmatrix}4 & 0 \\\\\\\\ 0 & 5\end{bmatrix}$，表示一个线性变换。变换后的新基向量分别是：
* $\hat{i} = \begin{bmatrix} 4 \\\\\\\\ 0 \end{bmatrix}$，也就是 $A$ 的第一列。
* $\hat{j} = \begin{bmatrix} 0 \\\\\\\\ 5 \end{bmatrix}$，也就是 $A$ 的第二列。

向量 $\vec{x}$ 在线性变换之后变成了 $\vec{v}$。

$\vec{v} = 2\hat{i}+ 1\hat{j}= 2\begin{bmatrix}4\\\\\\\\0\end{bmatrix} + 3\begin{bmatrix}0\\\\\\\\5\end{bmatrix} = \begin{bmatrix}8\\\\\\\\15\end{bmatrix}$

也就是说，矩阵 $A$ 代表的线性变换的作用是把一个向量在横轴上的长度拉长成原始的 4 倍，把纵轴上长度拉长成原始的 5 倍。

以上讲的都是 $A$ 是方阵的情况。如果 $A$ 不是方阵呢？

### 非方阵
❖  如果 $A$ 是一个 3×2 矩阵，表示什么样的线性变换？

答案是将一个 2 维面映射到一个 3 维空间里。举个例子：

矩阵 $\begin{bmatrix} 2 & 0 \\\\\\\\ -1 & 1 \\\\\\\\ -2 & 1 \end{bmatrix}$ 表示这样一个线性变换：把一个在 2 维面上的向量映射到 3 维空间中，这个 2 维面的基向量在 3 维空间中的坐标分别是 $\begin{bmatrix} 2 \\\\\\\\ -1 \\\\\\\\ -2 \end{bmatrix}$ 和 $\begin{bmatrix} 0 \\\\\\\\ 1 \\\\\\\\ 1 \end{bmatrix}$ 。

❖  如果 $A$ 是一个 2×3 矩阵，表示什么样的线性变换？

答案是将一个 3 维空间映射到一个 2 维面里。举个例子：

矩阵 $\begin{bmatrix} 3 && 1 && 4 \\\\\\\\ 1 && 5 && 9 \end{bmatrix}$ 表示这样一个线性变换：把一个在 3 维空间里的张量映射到 2 维面中，这个 3 维空间的基向量在 2 维面中的坐标分别是 $\begin{bmatrix} 3 \\\\\\\\ 1 \end{bmatrix}$ 和 $\begin{bmatrix} 1 \\\\\\\\ 5 \end{bmatrix}$ 和 $\begin{bmatrix} 4 \\\\\\\\ 9 \end{bmatrix}$ 。

❖  如果 $A$ 是一个 1×2 的矩阵，表示什么样的线性变换？

答案是将一个 2 维面映射到一个 1 维数轴上。举个例子：

矩阵 $\begin{bmatrix} 1 && 3 \end{bmatrix}$ 表示这样一个线性变换：把一个在 2 维面上的向量映射到 1 维数轴上，这个 2 维面的基向量在 1 维数轴上的坐标分别是 $\begin{bmatrix} 1 \end{bmatrix}$ 和 $\begin{bmatrix} 3 \end{bmatrix}$。

总结成一句话就是：和一个 m×n 的矩阵相乘代表从 n 维空间到 m 维空间的线性变换。或者说是 n 维原空间在 m 维空间的表达。m > n 则升维，m < n 则降维。

## Column Space（（矩阵的）列空间）
几何意义：

矩阵 $A$ 的列空间是指任一向量 $\vec{x}$ 经过 $A$ 线性变换之后的结果向量的集合。换句话说：矩阵 $A$ 的列空间是 $A\vec{x}$ 产生的所有向量张成（span）的空间。

如前所述，$\vec{x}$ 可以表示为 $ai + bj$，所以矩阵 A 的列空间也是 $A(ai + bj)$ 张成的空间。

## Determinant（行列式）
几何意义：
* 用来测量线性变换对空间的伸缩程度的度量。比如：在 2 维空间里测量面积变化，在 3 维空间里测量体积变化。
* Determinant($A$) = 0 表示线性变换 $A$ 会降维。比如 $\vec{x}$ 是一个 3 维张量, $\vec{v} = A\vec{x}$ 会是一个 2 维向量或 1 维点。此时 $\vec{x}$ 没有逆矩阵，即 $\vec{x}^{-1}$ 不存在。
* Determinant($A$) < 0 表示线性变化 $A$ 会改变方向。

举例（2 维面）：  
有一个向量 $\vec{x} = \begin{bmatrix}1\\\\\\\\1\end{bmatrix}$，它的起点、终点和横纵两轴合围而成的矩形的面积是 1。  
有一个矩阵 $A = \begin{bmatrix}3 & 0 \\\\\\\\ 0 & 4\end{bmatrix}$ 代表线性变换。  
线性变换之后的结果向量 $\vec{v} = A\vec{x} = \begin{bmatrix}3 & 0 \\\\\\\\\\\\\\\\ 0 & 4\end{bmatrix} \begin{bmatrix}1\\\\\\\\\\\\\\\\1\end{bmatrix} = \begin{bmatrix}3 \\\\\\\\\\\\\\\\     4\end{bmatrix}$，围起来的矩形的面积是 12。

那么 determinant($A$) = $12 \div 1 = 12$。表示经过线性变换 $A$ 之后，空间大小是原空间的 12 倍。

## Rank（秩）
代表经过线性变换之后空间（列空间）的维数。用来形容线性变换的维数变化。举例来说：
* 当 Determinant($A$) = 0 的维数变化情况。如果一个向量经过 $A$ 线性变换后，维数变为 2，那么我们就称矩阵 $A$ 的秩是 2。
* 如果线性变换前后的秩相同，则称 $A$ 是满秩的。而非满秩则代表线性变换 A 会降维。

## Dot Product（（2 维向量的）点积）

### 第一种几何意义
两个向量 $\vec{v}$ 和 $\vec{w}$ 的点积结果是其一在其二上的投影长度与其二长度的积，是一个标量。也就是说有这么两种计算点积的方法：
* 将 $\vec{v}$ 投影到 $\vec{w}$ 上的向量称为 $\vec{vp}$，那么 $\vec{v} \cdot \vec{w} = |vp| \times |w|$ 。
* 将 $\vec{w}$ 投影到 $\vec{v}$ 上的向量称为 $\vec{wp}$，那么 $\vec{v} \cdot \vec{w} = |v| \times |wp|$ 。

其结果可以有多种含义：
* 如果值是一个正数，表示 $\vec{v}$ 和 $\vec{w}$ 方向大致相同（夹角 < 90º）。
    * 值越大表示 $\vec{v}$ 和 $\vec{w}$ 的方向越相同，也就是夹角越小。
    * 值越小表示 $\vec{v}$ 和 $\vec{w}$ 的方向虽然大致相同、但差异越大，也就是夹角越大。
* 如果值是一个负数，表示两者方向大致相反（90º < 夹角 < 180º）。
    * 值越小（即绝对值越大）表示 $\vec{v}$ 和 $\vec{w}$ 的夹角越大。
    * 值越大（即绝对值越小）表示 $\vec{v}$ 和 $\vec{w}$ 的夹角越小。
* 如果值是零，表示两者垂直。

举例：
* $\vec{v} = \begin{bmatrix} 1 \\\\\\\\ 0 \end{bmatrix}$（在横轴上）
* $\vec{w} = \begin{bmatrix} 0 \\\\\\\\ 2 \end{bmatrix}$（在纵轴上）

两者点积 $\vec{v}\vec{w} = \begin{bmatrix} 1 \\\\\\\\ 0 \end{bmatrix} \cdot \begin{bmatrix} 0 \\\\\\\\ 1 \end{bmatrix} = 1 \times 0 + 0 \times 2 = 0$，说明两者垂直。

举例：
* $\vec{v} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix}$（在第一象限）
* $\vec{w} = \begin{bmatrix} 3 \\\\\\\\ 4 \end{bmatrix}$（在第一象限）

两者点积 $\vec{v}\vec{w} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix} \cdot \begin{bmatrix} 3 \\\\\\\\ 4 \end{bmatrix} = 1 \times 3 + 2 \times 4 = 11$，说明两者方向大致相同。

举例：
* $\vec{v} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix}$（在第一象限）
* $\vec{w} = \begin{bmatrix} -3 \\\\\\\\ -4 \end{bmatrix}$（在第三象限）

两者点积 $\vec{v}\vec{w} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix} \cdot \begin{bmatrix} -3 \\\\\\\\ -4 \end{bmatrix} = 1 \times -3 + 2 \times -4 = -11$，说明两者方向大致相反。

### 第二种几何意义
是对其中一个向量进行了从 2 维面到和另一个向量重叠的 1 维数轴的线性变换，结果是这个数轴上的点，用一个标量表示。

举例：
* $\vec{v} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix}$
* $\vec{w} = \begin{bmatrix} 3 \\\\\\\\ 4 \end{bmatrix}$

两者点积 $\vec{v} \cdot \vec{w} = \begin{bmatrix} 1 \\\\\\\\ 2 \end{bmatrix} \cdot \begin{bmatrix} 3 \\\\\\\\ 4 \end{bmatrix} = 1 \times 3 + 2 \times 4 = 11$

我们把 $\vec{v}$ 转换成一个 1×2 的矩阵并称之为 $A$。即 $A = \vec{v}^{-1} = \begin{bmatrix} 1 && 2 \end{bmatrix}$ 。  

矩阵 $A$ 和向量 $\vec{w}$ 相乘就是：$A\vec{w} = \begin{bmatrix} 1 && 2 \end{bmatrix}\begin{bmatrix} 3 \\\\\\\\ 4 \end{bmatrix} = 11$

可以看到“两个向量的点积结果”和“矩阵和向量相乘的结果”一样，所以可以把两个向量的点积看作进行了一次从 2 维面到 1 维数轴的线性变换（投射）。向量 $\vec{v}$ 也被称为这个线性变换在 2 维面中的对偶向量。

用另一句话来说就是，从一个 2 维面到 1 维数轴的线性变换一定能在 2 维面中找到一个对偶向量，使得这个线性变化（与矩阵相乘）等价于和这个对偶向量的点积。

## Cross Product（叉积）

3 维空间中的两个向量 $\vec{v}$ 和 $\vec{w}$ 的叉乘是一个新向量 $\vec{p}$：
* $\vec{p}$ 与 $\vec{v}$ 和 $\vec{w}$ 张成的 2 维面垂直，且方向符合右手定律（右手比成一个 3 维坐标轴的样子，用右手食指代表 $\vec{v}$ 的方向、中指代表 $\vec{w}$ 的方向，大拇指的方向就是 $\vec{p}$ 的方向）。
* $\vec{p}$ 的长度等于 $\vec{v}$ 和 $\vec{w}$ 围成的平行四边形的面积。


2 维面中的两个向量 $\vec{v}$ 和 $\vec{w}$ 的叉乘等于它们围成的平行四边形的面积（根据方向有正负）。

也等于它们组成的矩阵的行列式。令 $\vec{v} = \begin{bmatrix} a \\\\\\\\ b \end{bmatrix}$，$\vec{w} = \begin{bmatrix} c \\\\\\\\ d \end{bmatrix}$，它们的叉乘 $\vec{v} \times \vec{w} = determinant(\begin{bmatrix} a && c \\\\\\\\ b && d \end{bmatrix})$

（上面是讲叉积的第一个视频里的内容。第二个视频我没看懂）

## Eginvector & Eginvalue（特征向量和特征值）
特征向量的几何意义是：在经过一个线性变换之后得到的向量依然在它张成的空间里。

举例：

在 2 维空间里有这样一个向量 $\vec{v} = \begin{bmatrix} -1 \\\\ 1 \end{bmatrix}$，它张成的空间是一个数轴(出现在第二和第四象限、与横纵两轴形成的夹角都是 45º)。

有这样一个线性变换（矩阵） $A = \begin{bmatrix} 3 && 1 \\\\ 0 && 2 \end{bmatrix}$。

我们把 $\vec{v}$ 经过线性变换 $A$ 得到的向量称为 $\vec{w}$。$\vec{w} = A\vec{v} = \begin{bmatrix} 3 && 1 \\\\ 0 && 2 \end{bmatrix} \times \begin{bmatrix} -1 \\\\ 1 \end{bmatrix} = \begin{bmatrix} -2 \\\\ 2 \end{bmatrix} = 2\begin{bmatrix} -1 \\\\ 1 \end{bmatrix} = 2\vec{v}$。$\vec{w}$ 依然出现在 $\vec{v}$ 张成的数轴上，只是长度变成了 $\vec{v}$ 的 2 倍。

换句话说，对于在这个数轴上的任一向量，经过线性变换 A 之后的结果向量依然出现在该数轴上，只是长度拉伸为原来的 2 倍。

我们把具有这种特质的向量 $\vec{v}$ 称为特征向量，把 2 称为这个特征向量的特征值。每个特征向量都有一个所属的特征值。特征值衡量特征向量在变换中的伸缩比例，如果特征值是负数，表示变换后会转向。

除了特征向量的其他向量都会在线性变换之后离开之前张成的数轴。

用符号表示就是这样：$A\vec{v} = \lambda \vec{v}$。其中 $\vec{v}$ 是矩阵 $A$ 的特征向量，$\lambda$ 是 $A$ 的特征值。

但不是所有线性变换 $A$ 都有特征向量，比如 $A = \begin{bmatrix} 0 && -1 \\\\ 1 && 0\end{bmatrix}$ 这个矩阵代表的线性变换是逆时针旋转 90º，经过这个变换后所有向量都会旋转 90º、因而离开了自己在变换之前张成的数轴。所以这个 $A$ 没有特征向量。

在 3 维空间中，特征值为 1 的特征向量的几何意义是一个立方体的旋转轴（立方体是一个 3×3 的矩阵）。

有一种特殊矩阵是对角矩阵，其特殊性在于其所有基向量都是特征向量，其对角元是它们所属的特征值。这个基向量组合又被称为特征基。

与特征基相关的一个概念是：如果一个 N 维空间里的某个线性变换 $A$ 有 N 个足以张成 N 维空间的特征向量。那么我们就可以变换坐标系，这个新坐标系的基向量就是这 N 个特征向量。

比如：如果一个 2 维面里，线性变换 $A$ 有 2 个足以张成这个 2 维面的特征向量 $\vec{v}$ 和 $\vec{w}$，那么我们就可以变换这个 2 维面的坐标系，用这 $\vec{v}$ 和 $\vec{w}$ 作为基向量。

它的意义在于：如果一个向量或矩阵计算很复杂，那么可以尝试改变坐标系，让向量或矩阵在新坐标系的新描述形式：新向量或新矩阵，有可能对后者进行同样的计算就会容易很多。

举例：

有个矩阵 $A = \begin{bmatrix} 3 && 1 \\\\ 0 && 2 \end{bmatrix}$，我们要计算 $A^{10}$，这个计算很繁琐。而对角矩阵的次幂计算很容易。那我们就更换坐标系，让 $A$ 在新坐标系中是一个对角矩阵，然后再计算。具体方法是这样：
* Step 1：计算出 $A$ 的两个特征向量分别是 $\begin{bmatrix} 1 \\\\ 0 \end{bmatrix}$ 和 $\begin{bmatrix} -1 \\\\ 1 \end{bmatrix}$。
* Step 2：用两个特征向量组成一个矩阵 $E = \begin{bmatrix} 1 && -1 \\\\ 0 && 1 \end{bmatrix}$。
* Step 3：把 $A$ 用新坐标系来表述，称之为 $B$：$B = E^{-1} \times A \times E = \begin{bmatrix} 3 && 0 \\\\ 0 && 2 \end{bmatrix}$。
* Step 4：计算 $B^{10} = \begin{bmatrix} 3^{10} && 0 \\\\ 0 && 2^{10} \end{bmatrix}$。
* Step 5：把 $B^{10}$ 转换会原坐标系就是 $A^{10}$，即：$A^{10} = E \times B^{10} \times E^{-1}$ 。

用大白话来解释就是：如果有一个计算，对两个正交向量容易做，对不正交向量难做，那就通过变更坐标系，把这两个向量变成正交的，计算后再变更回原坐标系。
