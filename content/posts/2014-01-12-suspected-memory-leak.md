---
title           : "一起疑似内存泄漏"
date            : 2014-01-12
tags            : ["故障分析"]
category        : "研发"
isCJKLanguage   : true
---

上周遇到一起极其诡异的内存泄漏。现象是：  
某种测试方法可以使程序物理内存使用量（top命令的RES列）持续上升。当停止测试之后，程序内存使用量稳定在测试过程中的最高值不变。从现象上看就是一典型的内存泄漏。

我的诊断步骤：

1. 使用valgrind的memcheck检查泄漏。故障重现后让程序自然退出。memcheck提示资源泄漏量为0.  
   所以，该内存应该没有被泄漏，程序中对它们还有引用，在程序退出时被正常释放掉了。

2. 使用valgrind的massif分析内存使用。找到了一处可疑的「泄漏」点，90%的内存都是在该处分配。  
   但是调试发现该处分配（通过malloc）的内存都被正确的释放（通过free）掉了。

3. 使用malloc\_info函数查看程序运行过程中的内存情况。
   这是在程序物理内存占用2G后我停止测试以后一段时间（大概5分钟）的输出：

     arena(462848), ordblks(2), smblks(1), hblks(5), hblkhd(104857600), usmblks(0), fsmblks(32), uordblks(459424), fordblks(3424), keepcost(3360).

   从上面看，从heap中申请了462848(arena的值，约46KB)，mmap申请的内存是104857600（hblkhd的值，约100MB）。这俩值加在一起和top里显示的2G差很远啊。
   注，arena和hblkhd的man注解：

     arena   The total amount of memory allocated by means other than
             mmap(2) (i.e., memory allocated on the heap).  This figure
             includes both in-use blocks and blocks on the free list.
     hblkhd  The number of bytes in blocks currently allocated using
             mmap(2).
4. 决定统计内存分配释放的次数和尺寸，目的是检查是否严重失衡。因为new/delete不太好跟踪，所以只处理了malloc/free。  
   使用的方法是wrap（gcc/g++的「-Wl,--wrap,malloc」选项），看了一眼glibc的源码，以在free时候得到待释放内存的尺寸。最后的结果是没有严重失衡，应该不是显式的内存释放。  
   其实这个实验原本就没有报很大的希望，一来new/delete没有照顾到，不够精确。二来如果是显式的泄漏memcheck早就报告了。但是当时没有思路了，姑且一试，看能不能找到点面包屑。

5. 搜索了一些资料。有些提到glibc内部使用brk和mmap来进行内存分配，其中brk可能会造成这种泄漏假象。  
   于是在程序的开始设置了M\_MMAP\_THRESHOLD，发现故障依然存在。

     mallopt(M_MMAP_THRESHOLD, 1024*1024*1024); // 1G

原本还想在不同的模块里使用不同的堆，然后反复根据堆的尺寸细分堆，来精确定位。却发现linux下没有类似HeapCreate的函数。

技穷了。于是我邀请一位同事来协助分析，在协作中我发现我犯了个错误：尝试方法5的时候，我理解反了M_MMAP_THRESHOLD的含义，于是提供错了参数。实际上应该是这样：
 
 mallopt(M_MMAP_THRESHOLD, 0);

然后故障消失了。也就是说，这次的故障实际并不是我们程序实现的资源泄漏，而是glibc堆管理机制导致的「资源泄漏」。我很奇怪为什么glibc没有重用程序调用free释放掉的空间，而是继续扩充堆。这个问题值得继续琢磨。

总结：本次故障的分析过程里我的粗心大意又给自己惹了大麻烦。原本思路是正确的，结果走歪了。。。 自作孽啊。。。

但是，即便这个故障不是程序实现BUG所导致的，实际上还是会对程序造成不好的影响：

* 如果调整M_MMAP_THRESHOLD让程序不「泄漏」，那么资源占用和性能会变差。原因参考帮助文档中对该参数的描述。
* 如果放任不管，程序会不停地吃内存，并且占住不释放。我尝试让程序持续吃内存，最后稳定在3GB的物理内存（共4G）和2.6G的swap空间（共4G）的消耗。（但没有被oom killer干掉）

再琢磨琢磨吧。

最后，感谢我的同事cntrump。并附上《代码大全》里的一段话，我前些天重温的时候才看过：

>23.2 __寻找缺陷__ - _同他人讨论问题_  
>　　有人会把这种方法称之为“忏悔式调试”。当你向别人解释自己的程序时，常常能发现自己犯下的一些错误。举个例子，如果你向别人解释上面的关于薪水的例子，你或许会这样对别人说：  
>　　嗨，jennifer，你有空么？我现在遇到一个麻烦。这张员工薪水列表本来应当是按照顺序排列的，但里面有些名字乱序了。我原本打算看看是不是新输入的名字就会这种情况，但有时是对的，有时又不是。我向这些数据在我输入他们的时候就应当被排序，因为程序会在我输入数据的时候对其排序，然后在数据保存的时候再排一遍。等一下，不对，它没有在输入数据的时候对其排序。就是这里。程序只是粗略地对这些数据进行了排序。谢谢你，jennifer，你帮了我个大忙。

