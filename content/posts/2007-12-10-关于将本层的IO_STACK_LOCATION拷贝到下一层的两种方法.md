---
title           : 关于将本层的IO_STACK_LOCATION拷贝到下一层的两种方法
date            : 2007-12-10
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

两种方法：

1. 调用MS提供的标准方法IoCopyCurrentIrpStackLocationToNext(...)
2. 方法二：
{{< highlight cpp >}}
PIO_STACK_LOCATION IrpSp;   
PIO_STACK_LOCATION NextIrpSp;   
  
IrpSp = IoGetCurrentIrpStackLocation(Irp);   
NextIrpSp = IoGetNextIrpStackLocation(Irp);   
  
*NextIrpSp = *IrpSp;   
  
return IoCallDriver(NextDeviceObject, Irp);
{{< /highlight >}}  

两种方法都挺常用。但是今天再看DDK中一篇OSR的分析文章里提到一个采用方法二可能会导致的一个很隐蔽的BUG：  

方法二把本层（La）的整个IO_STACK_LOCATION都拷贝到了下一层(Lb)，而IO_STACK_LOCATION中有两个成员CompletionRoutine、Context，即完成例程和完成例程的上下文参数。也就是说方法二会让Lb层拥有这两个原本不一定会属于它的成员。如果这两个成员都是NULL，那还好。一旦这两个成员有有效的内容，那么就会导致"an eventual blue screen"。

再反观方法一IoCopyCurrentIrpStackLocationToNext(...)，在Ntddk.h有，有这个“函数”的定义：
    
{{< highlight cpp >}}
#define IoCopyCurrentIrpStackLocationToNext( Irp ) { \   
PIO_STACK_LOCATION __irpSp;        \   
PIO_STACK_LOCATION __nextIrpSp;    \   
__irpSp = IoGetCurrentIrpStackLocation( (Irp) );     \   
__nextIrpSp = IoGetNextIrpStackLocation( (Irp) );    \   
RtlCopyMemory(__nextIrpSp, __irpSp, FIELD_OFFSET(IO_STACK_LOCATION, CompletionRoutine)); \     
__nextIrpSp->Control = 0; }
{{< /highlight >}}

原来这是个宏，重点看RtlCopyMemory调用的最后一个参数：这个宏只拷贝了CompletionRoutine成员之前的部分。方法一、方法二的区别就在这里。而实际上后者相对于前者，并没有什么优点，所以还是尽量使用方法一比较好。
