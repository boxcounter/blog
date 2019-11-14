---
title:  Android下的MVP开发模式
date:   2016-01-27
draft:  true
category: note
---


如题所述，这是一篇记录我在Android开发中对MVP模式的思考。MVP相关资料数不胜数，Android下的MVP开发博文也不少。但多是讲what、how，甚少提及why。所以在实际应用过程中，很容易出现误解和走形。所以我认为写下这篇思考记录是有意义的，而非单纯的造车轮子。

# MVP开发模式的定义

先来看看[维基百科上MVP的组件职责](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter#Pattern_description)

- The **model** is an interface defining the data to be displayed or otherwise acted upon in the user interface.
- The **presenter** acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view.
- The **view** is a passive interface that displays data (the model) and routes user commands (events) to the presenter to act upon that data.

看似各组件职能划分地很干净，但太『虚』。怎么把上述三组件的划分应用到手头上的项目里呢？我猜有部分同好会感觉不知道从何下手，另有部分同好会觉得方案很多，不确定哪种才复合纯粹的MVP模式。

其实，不管是哪一种情况，背后的实质都是因为这种教材式的MVP介绍太抽象。也因为此，不管是MVC还是MVP，现如今都有很多变种。

我希望能通过这篇博文让MVP开发模式能实实在在的落在Android下的常见开发场景中。但因为个人能力原因所限，所述内容仅供同好参考，非常欢迎一起讨论。

# 讲解环境介绍

我们先来看一个很简单的Android应用：只有一个MainActivity，其布局上只有一个Button，点击它会发送一条消息（给某个朋友，具体细节可以忽略）。

按照主流的Android下MVP的组件划分，应该会分成这几样：

- Modal: 新建的一个MainActivityModal类，包含send()方法；
- Presenter: 新建的一个MainActivityPresenter类。从View接受用户响应并将具体的发送（业务逻辑）交给Modal完成；
- View: 就是MainActivity。负责设置button的click listener，在其中调用Presenter的方法；

乍一看上，好像模块划分的很标准，但这只是第一步，很多细节问题都没有暴露出来。我们来看看几个常见问题。

# 问题1

问题1：Presenter对View暴露的什么层次的方法。比如：当用户点击Button时，View应该调用Presenter.sendMessage()还是Presenter.handleButtonClick()？（同好可以在这里停顿一下，想一想你会怎么选择，为什么这么选择）

我的做法：View调用Presenter.handleButtonClick()。

我的思路：

1. 从click到sendMessage，这个过程实际上是一种业务逻辑判定。而View应该只负责UI相关的**展现**（为什么要强调？）功能，不负责逻辑。所以P应该提供的时handleButtonClick()（为什么不叫onButtonClick？）；
2. 当对交互有调整时，P暴露handleButtonClick可以更好的重用。比如，现在改为点击Button后需要弹出确认提示框，用户点击确认后才发送。如果P暴露的时sendMessage方法，那么弹出dialog、响应『确认』、『取消』的代码就需要放在View中。而MVP中，P是重用度比V高很多，经常出现一个P服务于多个不同的V，所以把dialog相关逻辑放在P中有更好的可重用性（dialog是UI组件，这样安排岂不是把一部分UI展示功能分散到了P中？）；

到这里，部分同好可能有些晕了。没关系，以上是纯文字讲解，只是为了简要说明问题。大家可以发现，MVP组件的职能划分在高度抽象时很容易做到，但一旦要落到具体项目逻辑实处，就遇到细节问题。但追根溯源，困难集中在两点：各个组件到底负责什么？怎样设定组件之间的接口（抽象名词，并不是指java关键字interface）？

在这里我先提出我的回答：

职能划分：

- M: 负责『业务逻辑执行』。比如调用网络库（如volley）发送请求；
- P: 负责『交互逻辑』和『业务逻辑分派（dispatch）』；
- V: 负责『交互』；

这里需要解释两个关键词定义：

- 交互逻辑：P收到V传来的交互请求后（比如用户点击Button），继续进行的交互操作流程（比如弹出dialog）；
- 业务逻辑分派：P收到V传来的交互请求后，如果判定这个交互请求所对应的是业务逻辑（比如用户点击了dialog的『确定』按钮），就转交给Modal处理；

如果套用回样例工程，应该是这样：

- MainActivityModal：提供send()方法，调用网络库（比如volley）发送请求到服务器；
- MainActivityPresenter：提供handleButtonClick()方法，弹出dialog，并设置『确定』按钮的响应方法是调用MessageModal.send()；
- MainActivity：设定Button的ClickListener，在其中调用MainActivityPresenter.handleButtonClick()；

前面我提出了一个问题：如果把dialog相关逻辑放在Presenter中，岂不是让Presenter也负责了交互？

没错，确实有这个问题。如果参照前面的职能划分，只有V负责交互。怎么办？答：把dialog逻辑拆分为『交互』和『业务逻辑分派』，前者放回到V，后者留在P。

我们来看一下代码，当点击Button就直接发送消息时，代码是这样的：

```java
public class MainActivityModal {
    public void send(String text) {
        // TODO
    }
}

public class MainActivityPresenter {
    private MainActivityModal mModal;

    public void handleButtonClick(final String text) {
        mModal.send(text);
    }
}

public class MainActivity {
    private MainActivityPresenter mPresenter;

    private void setUpSendButton() {
        Button button = (Button) findViewById(R.id.btn_send);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPresenter.handleButtonClick("boxcounter.com");
            }
        });
    }
}
```

当点击Button需要弹出确认提示框时，代码是这样的：


```java
public class MainActivityModal {
    public void send(String text) {
        // TODO
    }
}

public class MainActivityPresenter {
    private MainActivity mView;
    private MainActivityModal mModal;

    public void handleButtonClick(final String text) {
        mView.showQueryDialog(R.string.alert_send, new Runnable() {
            @Override
            public void run() {
                mModal.send(text);
            }
        });
    }
}

public class MainActivity {
    private MainActivityPresenter mPresenter;

    @Override
    public void showQueryDialog(@StringRes int messageResId, final Runnable positiveHandler) {
        new AlertDialog.Builder(this)
                .setMessage(messageResId)
                .setPositiveButton(R.string.confirm, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                        positiveHandler.run();
                    }
                })
                .setNegativeButton(R.string.cancel, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                })
                .show();
    }

    private void setUpSendButton() {
        Button button = (Button) findViewById(R.id.btn_send);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPresenter.handleButtonClick("boxcounter.com");
            }
        });
    }
} 
```

通过这样的拆分，M、V、P就可以各尽其职了。



# 问题

- Fragment、Activity、Contenxt污染Presenter
- Activity/Fragment启动的方式（ShareElement）、参数不同。当一个View的多个child对应着启动不同的Activity，怎么交给View来处理;

# 重新整理

# Android的设计缺陷


