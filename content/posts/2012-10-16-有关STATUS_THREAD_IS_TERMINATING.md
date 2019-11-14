---
title           : 有关STATUS_THREAD_IS_TERMINATING
date            : 2012-10-16
tags            : ["故障分析"]
category        : "研发"
isCJKLanguage   : true
---

　　手头有个驱动，因为客户需求的原因，只在x86的2k3和xp系统上做过测试，今天在把它往x64 win7系统移植的时候遇到这么一个问题：  
　　在PsSetCreateProcessNotifyRoutine注册的回调函数中，通过FltSendMessage发通知给应用层接收者。在发送“进程退出”事件时FltSendMessage总是会返回STATUS_THREAD_IS_TERMINATING（0xC000004B）。

　　琢磨了一会，找到了问题关键，分析过程如下：

1. 定位返回STATUS\_THREAD\_IS\_TERMINATING的点

        1: kd> pc
        fltmgr!FltSendMessage+0x1bf:
        fffff880`011a1ddf call    qword ptr [fltmgr!_imp_FsRtlCancellableWaitForMultipleObjects (fffff880`011a75c8)]
        
        1: kd> p
        fltmgr!FltSendMessage+0x1c5:
        fffff880`011a1de5 mov     ebx,eax
        
        1: kd> r eax
        eax=c000004b
   这里定位到在FltSendMessage函数中是FsRtlCancellableWaitForMultipleObjects这个函数返回的这个错误。
2. 继续更细致的定位

        ...
        nt! ?? ::NNGAKEGL::`string'+0xcb90:
        fffff800`0418d031 mov     rax,qword ptr gs:[188h]
        fffff800`0418d03a mov     ecx,dword ptr [rax+448h]
        fffff800`0418d040 test    cl,1
        fffff800`0418d043 jne     nt! ?? ::NNGAKEGL::`string'+0xcc16 (fffff800`0418d0b7)
        ...
        nt! ?? ::NNGAKEGL::`string'+0xcc16:
        fffff800`0418d0b7 mov     eax,0C000004Bh
        fffff800`0418d0bc jmp     nt!FsRtlCancellableWaitForMultipleObjects+0x69 (fffff800`0416a271)
        ...
   很幸运，就是FsRtlCancellableWaitForMultipleObjects这个函数返回的STATUS\_THREAD\_IS\_TERMINATING。
   （有时候要深入好几层函数调用才能定位到错误码的直接返回者）
3. 根据2中的汇编码，继续分析

        1: kd> dp gs:[188h]
        002b:00000000`00000188  fffffa80`0d168b60 
        
        
        1: kd> !pool fffffa80`0d168b60  2 
        Pool page fffffa800d168b60 region is Nonpaged pool
        *fffffa800d168b00 size:  500 previous size:  280  (Allocated) *Thre (Protected)
        	    	Pooltag Thre : Thread objects, Binary : nt!ps
   看来这个地址是个线程结构体。
   为了确认这个地址就是线程结构体的开头，再找个线程相关函数来看看吧：

        0: kd> u nt!KeGetCurrentThread
        nt!PsGetCurrentThread:
        fffff800`03f23f40 mov   rax,qword ptr gs:[188h]
        fffff800`03f23f49 ret
   那么不是kthread就是ethread，而汇编码中取值的偏移是0x448，前者大小不对，那看看后者的0x448是什么吧

         1 1: kd> dt nt!_ethread
         2 nt!_ETHREAD
         3    ...
         4    +0x448 CrossThreadFlags : Uint4B
         5    +0x448 Terminated       : Pos 0, 1 Bit
         6    +0x448 ThreadInserted   : Pos 1, 1 Bit
         7    +0x448 HideFromDebugger : Pos 2, 1 Bit
         8    +0x448 ActiveImpersonationInfo : Pos 3, 1 Bit
         9    +0x448 Reserved         : Pos 4, 1 Bit
        10    +0x448 HardErrorsAreDisabled : Pos 5, 1 Bit
        11    +0x448 BreakOnTermination : Pos 6, 1 Bit
        12    +0x448 SkipCreationMsg  : Pos 7, 1 Bit
        13    +0x448 SkipTerminationMsg : Pos 8, 1 Bit
        14    +0x448 CopyTokenOnOpen  : Pos 9, 1 Bit
        15    +0x448 ThreadIoPriority : Pos 10, 3 Bits
        16    +0x448 ThreadPagePriority : Pos 13, 3 Bits
        17    +0x448 RundownFail      : Pos 16, 1 Bit
        18    +0x448 NeedsWorkingSetAging : Pos 17, 1 Bit
        19 	...
   对照着汇编码，可以知道在FsRtlCancellableWaitForMultipleObjects函数判断了当前线程的ethread::Terminated（行5）这个域，而在我的调试环境中该域是1，于是FsRtlCancellableWaitForMultipleObjects就返回了STATUS\_THREAD\_IS\_TERMINATING。

　　到这里，已经弄清楚了在什么情况下会出现这个错误。那解决方案也很简单，不赘述。

　　另外，多说两句：

1. 这个特性是从vista开始引入。
2. 目的是为了保证线程结束的处理过程中不被卡住。


　　参考资料：

* [ STATUS_THREAD_IS_TERMINATING returned by FltSendMessage](http://www.osronline.com/showThread.cfm?link=110848)
