---
title:  Android开发模式之二：MVP模式
date:   2016-06-09
draft: true
category: technique
---

## 前言


本文是《Android开发模式》系列的第二篇，主要介绍MVP模式。其它几篇也罗列在这里，方便同好查阅。

* [《Android开发模式之一：传统模式》](http://localhost:1313/technique/2016-06-09-Android%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B9%8B%E4%B8%80%E)
* 《Android开发模式之二：MVP模式》
* [《Android开发模式之三：常见问答》](http://localhost:1313/technique/2016-06-09-Android%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B9%8B%E4%B8%80%EF%BC%9A%E4%BC%A0%E7%BB%9F%E6%A8%A1%E5%BC%8F/)
* [《Android开发模式之三：最佳实践》](http://localhost:1313/technique/2016-06-09-Android%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B9%8B%E4%B8%80%EF%BC%9A%E4%BC%A0%E7%BB%9F%E6%A8%A1%E5%BC%8F/)

## MVP的特点

MVP模式有很多优点，给我留下深刻印象的是：

1. 解耦
2. 单一职责
3. 高可复用性
4. 易于交流

Model/View/Presenter三组件结构就决定了特点1、2、3，从前述手工打造的MVC也可见一斑。而特点4和设计模式是相似的：

> Having a vocabulary for patterns lets us talk about them with our colleagues,  
> in our documentation, and even to ourselves. It makes it easier to think about  
> designs and to communicate them and their trade-offs to others. Finding good  
> names has been one of the hardest parts of developing our catalog.
>
> ——《Design Patterns》

但也有缺点：

1. 学习成本
2. class、interface的数量增加

## 职能划分

### View

1. 响应交互
    - 接收用户的交互事件，并转交给Presenter处理。
    - 接受Presenter的指令（调用），完成相应的UI呈现。
2. 响应生命周期事件。
3. （容器View）管理（增、删、改）Sub views。

### Presenter

1. 交互逻辑。从View接受交互事件，判断、选择相应的处理方案。比如：需要弹出对话框吗？需要跳转到另一个Activity吗？
2. 调用Model执行业务逻辑。
3. （容器的Presenter）管理（增、删、改）Sub presenters。
4. （容器的Presenter）分派逻辑给Sub presenters。

### Model

1. 业务逻辑。比如：发起网络请求并解析响应。
2. 数据的维持。比如：本地存储的维护、内存数据的增删查改。
3. 为Presenter提供信息支撑。

