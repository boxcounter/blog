---
title           : "baselineAligned属性"
date            : 2015-09-01
tags            : ["Android"]
category        : "研发"
isCJKLanguage   : true
---


在使用LinearLayout+TextView的过程中遇到一个看似诡异的问题，显示效果如下：

<img src="/images/2015-09-01/baselineAligned.png" width="300"/>

其中两个灰色方块是使用代码动态创建的TextView。它们的父LinearLayout是这样的：

{{< highlight xml >}}
<LinearLayout
    android:id="@+id/ll_files"
    android:layout_width="wrap_content"
    android:layout_height="100dp"
    android:layout_marginTop="7.5dp"
    android:orientation="horizontal"
    />
{{< /highlight >}}

不知道是什么原因，导致第一个TextView「下挫」了一些。仔细检查了它的margin和padding都是0、符合预期，但top是29。

仔细阅读了一遍代码和布局文件，确定我并没有误调整top，那么问题应该出现在动态layout过程中。那就开始调试分析吧。

顺着LinearLayout.onLayout()追踪到了LinearLayout.layoutHorizontal()，发现了这样一段代码：

{{< highlight java >}}
void layoutHorizontal(int left, int top, int right, int bottom) {
    // 若干无关代码，略
    
    int childBaseline = -1;
    
    final LinearLayout.LayoutParams lp =
            (LinearLayout.LayoutParams) child.getLayoutParams();
    
    if (baselineAligned && lp.height != LayoutParams.MATCH_PARENT) {
        childBaseline = child.getBaseline();
    }
    
    // 若干无关代码，略
    
    switch (gravity & Gravity.VERTICAL_GRAVITY_MASK) {
        case Gravity.TOP:
            childTop = paddingTop + lp.topMargin;
            if (childBaseline != -1) {
                childTop += maxAscent[INDEX_TOP] - childBaseline;  // <==== A
            }
            break;
    
    // 若干无关代码，略
}
{{< /highlight >}}


当执行完A行时，childTop等于29。问题出在这里，再反向阅读代码，跟踪到了两个关键变量：

1. baselineAligned为true；
2. childBaseline不为0；

这时我想起文本的[baseline属性](https://zh.wikipedia.org/wiki/%E5%9F%BA%E7%B7%9A)。

继续查阅文档、代码，得到两个结论：

1. LinearLayout的baselineAligned默认为true；
2. TextView重写了getBaseline()，返回了有效值；

定位了问题症结，解决方案也就不难找了：将LinearLayout的android:baselineAligned改为false。

