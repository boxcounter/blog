---
title           : "依赖倒置原则（Dependency inversion principle）"
date            : 2019-06-02
category        : "研发"
isCJKLanguage   : true
---

依赖倒置原则（**D**ependency **I**nversion **P**rinciple）是 Uncle Bob 提出的一个 OOP 的原则。维基百科上对它的说明是这样的：

> 该原则规定：  
> 1. 高层次的模块不应该依赖于低层次的模块，两者都应该依赖于抽象接口。  
> 2. 抽象接口不应该依赖于具体实现，而具体实现则应该依赖于抽象接口。  

与上述文字相比，我更喜欢用的图和伪码来描述。来看一下最常见的依赖关系：直接依赖 —— 依赖于具体实现。如下图一：

<img src="/images/2019-06-02/01.png"/>

而 DIP 更推荐下图二这种结构：间接依赖 —— 依赖于抽象接口。

<img src="/images/2019-06-02/02.png"/>

伪码和之前几乎一样，但依赖关系发生了变化：Foo 依赖于同包的接口 Bar，而不是另一个包的实现类。这样做的实质是分离了依赖关系和调用关系：

- 图一中，Foo 既依赖了 Bar，又调用了 Bar。
- 图二中，Foo 仍依赖了 Bar，但调用的是 ConcreteBar（Foo 所需的功能由 ConcreteBar 实际提供）

那我们为什么需要将依赖关系和调用关系分离呢？为了降低耦合。

我们来看一个常见的业务场景：后端响应用户的登录请求。结构如下图三：

<img src="/images/2019-06-02/03.png"/>

在实现时，我们将结构分为两层：

- 应用层 —— 负责处理业务逻辑。对应图中的 Application 包。
- 基础设施层 —— 负责技术组件的具体实现。对应图中的 Infrastructure 包。

此外，还用到了 Repository 设计模式：图中的 UserRepository 负责从 MySQL 中读取用户信息。

图中的结构和伪码简单明了，而且很符合一些开发者的编写习惯。但它有一些缺陷。

假设现在需要把一部分热数据缓存在 Redis 中以降低 MySQL 的查询压力。你会怎么做？

常见的可能有这么两种：

方案一：修改 UserRepository 类，让它在负责操作 MySQL 之外也负责操作 Redis。如图四：

<img src="/images/2019-06-02/04.png"/>

这样的好处是应用层代码不需要修改，技术变更被控制在负责技术实现的基础设施层。

那么它有缺点吗？有的，比如让 UserRepository 的职责变多、且（操作 MySQL 和 操作 Redis）耦合在一起。

那么这个会有副作用吗？是的，会让我们很难对它的某个职责进行针对性处理。

比如，现在我们需要单独对 MySQL 进行压力测试。那么为了屏蔽 Redis 功能，我们不得不在 UserRepository 中增加大量的 `if (redisEnabled) {...` 这样的判断语句，这会让 UserRepository 的代码结构格外的庞杂和凌乱，因为它承担了三个截然不同的职责：

1. 操作 MySQL。
2. 操作 Redis。
3. 指挥 Redis 和 MySQL 之间的协同（比如处理 Redis 未命中数据请求时的逻辑）。

让我来看看另一种常见做法。

方案二：增加 UserCache 类，专门负责操作 Redis。如下图五：

<img src="/images/2019-06-02/05.png"/>

这样的解决了方案一的缺点：他将方案一中提到的三种职责分到了三个不同的地方：

1. 操作 MySQL：由 UserRepository 承担。
2. 操作 Redis：由 UserCache 承担。
3. 指挥 Redis 和 MySQL 之间的协同：由 UserApplicationService 承担。

但这个方案的缺点是让技术变更渗透到了应用层：UserApplicationService 的诉求是获得 User 数据，至于这个数据来自 Redis 还是 MySQL，它并不关心。毕竟它的职责是处理业务逻辑，而非技术细节。

到这里我们会发现，上述两种方案的优缺点正好是相反的。我们是否只能在这两者中做“两害相权取其轻”的抉择呢？当然不是，我们来看看第三种方案。

方案三：使用 DIP 原则分离业务逻辑和技术实现。

我们先看看引入 Redis 前的结构，如图六。

<img src="/images/2019-06-02/06.png"/>

此时 UserApplicationService 依赖的是 UserRepository 这个抽象接口（而非具体实现类），而 MySQLUserRepository 则实现了这个抽象接口。

引入 Redis 后结构会变成图七的样子。

<img src="/images/2019-06-02/07.png"/>

这个结构同时囊括了前述两个方案的优点：

1. 技术变更的影响被在控制基础设施层内，应用层代码无需修改。
2. 三个职责由三个不同的类承担：
	1. 操作 MySQL：由 UserRepository 承担。
	2. 操作 Redis：由 UserCache 承担。
	3. 指挥 Redis 和 MySQL 之间的协同：由 CompositeUserRepository 承担。

而能够同时做到这些的原因就是我们应用了 DIP 原则：

1. 高层次的模块（UserApplicationService）不应该依赖于低层次的模块（XXXUserRepository），两者都应该依赖于抽象接口（UserRepository）。
2. 抽象接口（UserRepository）不应该依赖于具体实现（XXXUserRepository），而具体实现则应该依赖于抽象接口。

从本质上讲，DIP 分离了依赖关系和调用关系、并改变了依赖方向。从而通过实现“调用但不依赖”的关系结构，来降低层与层、模块与模块、类与类之间的耦合。

到这里，DIP 原则的主要内容就讲解完毕，但 DIP 的作用并不仅限于此。使用 DIP 原则降低耦合之后会让开发者做很多事情都如虎添翼，比如单元测试。如果感兴趣，可以思考一下如何为图一和图六中的 UserApplicationService 编写单元测试，相信会体会到更多低耦合带来的畅快感受。

另外，除了 DIP，Uncle Bob 提出的其他几个原则对作出优雅的设计也很有益处。如果你对这篇文章感兴趣，也许你也会喜欢 Uncle Bob 的著作《敏捷软件开发：原则、模式与实践》。
