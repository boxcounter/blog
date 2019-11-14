---
title           : 关于完成例程（Completion Routine）
date            : 2011-10-26
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

通常使用完成例程有这么三步：

1. 调用IoCopyCurrentIrpStackLocationToNext(...)函数，把当前的IRP栈数据复制一份到下一层。
2. 调用IoSetCompletionRoutine(...)函数，为IRP设置完成例程。
3. 调用IoCallDriver(...)函数，把IRP传递到下一层驱动对象。

DDK对使用完成例程还有如下要求：

1. 与'IRQL'相关的要求(因为完成例程有肯能运行在DISPATCH_LEVEL级，所有对IRQL有要求)：
    1. 对于要求运行在低IRQL上的部分内核函数(如IoDeleteDevice、ObQueryNameString), 完成例程无法很安全的调用它们。
    2. 完成例程使用的数据结构必须从非分页内存中分配。DDK上关于前面提到的“数据结构”讲述很模糊，像是完成例程的context参数。不过我觉得，除了在栈上分配的空间外，完成例程中所有的内存分配都应该在非分页堆中分配。
    3. 完成例程不能放在分页内存中。
    4. 完成例程不能'申请资源（acquire resources）'、'mutexts'、'fast mutexes'。但是可以申请自旋锁。（我没有理解'申请资源'具体指的是什么）

2. 检查PendingReturned标志（这一方面，DDK上说的很清楚，就不画蛇添足翻译了。个人感觉MS在MSDN上真是费了不少心血，绝大部分的句子都是用的最基本的语法，而且描述的很清楚）
    1. Unless the completion routine signals an event, it must check the Irp->PendingReturned flag. If this flag is set, the completion routine must call IoMarkIrpPending to mark the IRP as pending. 
    2. If a completion routine signals an event, it should not call IoMarkIrpPending.

3. 返回值：  
过滤驱动的完成例程只能有两种返回值：'STATUS_SUCCESS'，或者'STATUS_MORE_PROCESSING_REQUIRED'。其他任何的返回值都会被IO manager重置为'STATUS_SUCCESS'。

4. 把IRP包放到work queue中应注意的：  
先调用IoMarkIrpPending(...)，再把IRP包放到queue中。如果顺序反了，则在两者调用的之间的那段时间，该IRP包可能已经被其他驱动处理函数从queue中拿出，处理完毕，并且释放掉了该IRP包。这种情况下，可能导致系统崩溃。
还有一些关于完成例程在哪些情况下，应该返回'STATUS_MORE_PROCESSING_REQUIRED'。这里就不罗嗦了，具体看DDK中“Constraints on Completion Routines”这一节。
