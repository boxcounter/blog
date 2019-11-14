---
title:  Android传统开发模式与MVP模式的实践与思考
date:   2016-06-09
draft: true
category: technique
---

## 前言

作为Android开发者，最熟悉的开发模式自然是一个Application加上若干个Activity。每个Activity中，顺着其生命周期方法安排相应的功能。典型的就是：

- 在onCreate()中初始化各种View，包括设定Listener，在其中安排逻辑代码；
- 在onSaveInstance()中保存各种数据；

这种模式就是俗称的『传统开发模式』。随着Android开发者群体的日益壮大，项目规模日渐庞大，传统开发模式的局限也逐渐体现出来，越来越多的开发者开始寻找更优的Android软件构建模式。这时，桌面开发时代就盛行的MVC、MVP、MVVM模式自然成为了众多开发者首先尝试的解决方案。

本文并不准备完整的介绍MVC、MVP、MVVM这三种模式的详细内容与区别。而是讲重点放在如何用MVP模式来解决因传统开发模式的局限性导致的问题。


## 名词定义

本文中会反复提到几个名词，为了方便区分，在这里先定义它们的概念。

- 交互逻辑：泛指UI相关的判定、执行逻辑。比如：某个View是否enable、是否visible、变更enable、变更visibility，显示Toast，弹出Dialog；
- 业务逻辑：泛指与项目业务相关的逻辑。比如：发起网络请求创建一个用户、发表一个评论；
- Entity：包含属性、相应Setter/Getter、以及对其属性进行简单操作的方法的数据类、对象；

有了这些预热工作，咱们可以开始进入正题了。首先我们需要明确传统开发模式的问题。


## 传统开发模式的问题

传统开发模式的所有代码都放在Activity/Fragment中（为行文简洁，后简称A/F），比如：UI、交互逻辑、业务逻辑、生命周期、权限申请/管理。

这会带来以下几种问题：

**1. 可读性性差**

A/F代码量大，职能庞杂，导致代码可读性变差。这个是开发者的共识，无需多言。

**2. 可复用性差**

当多个A/F引用了相同的UI组件时，常见的做法是：

    实现一个Custom View类，接受一个Entity，将其呈现。同时对外提供一个Listenr/Callback（为行文简洁，后简称为L/C） interface。每个A/F创建自己的实现。

这解决了View或者说是UI的复用问题。但并没有解决相应的交互逻辑和业务逻辑的复用问题：每个A/F都需要实现一个负责交互逻辑和业务逻辑的L/C。

于是，为了实现逻辑的可复用性，常用的做法是：

    实现一个L/C类，每个A/F都创建一个实例。（Entity会作为参数传递给它，供逻辑代码使用）

到这里，我们审视这个解决方案，会发现这实际上就是一个手工打造的、粗糙的MVC模式：

- Entity是Model；
- L/C是Controller；
- Custom View是View；

**3. 无法（单元）测试**

因为所有的**单元**都融进了F/A，相互交杂，没法针对单个单元做测试。最多只能进行集成测试和UI测试。

而根据Mike Cohn的测试金字塔概念，单元测试、集成测试、UI测试之比应为7:2:1。

** TODO 此处需要有图 **

所以，传统开发模式基本无法测试。

而无法测试意味着难以在保证稳定性的前提下进行维护和扩展（具体原因留待以后的测试文章讲述）。

所以，开发者们把希望的目光投向了MVPMVP。


## MVP模式

### 特点

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

### 职能划分

**View**

1. 响应交互
    - 接收用户的交互事件，并转交给Presenter处理。
    - 接受Presenter的指令（调用），完成相应的UI呈现。
2. 响应生命周期事件。
3. （容器View）管理Sub Views。

**Presenter**

1. 交互逻辑

**Model**

1. 业务逻辑。比如：发起网络请求并解析响应。
2. 数据的维持。比如：本地存储的维护、内存数据的增删查改。
3. 为Presenter提供信息支撑。

