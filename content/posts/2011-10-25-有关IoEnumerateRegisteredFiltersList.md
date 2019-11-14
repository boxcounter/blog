---
title           : "有关IoEnumerateRegisteredFiltersList"
date            : 2011-10-25
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

有关于IoEnumerateRegisteredFiltersList
昨天逆一个驱动的时候发现静态分析出来的逻辑跟动态调试的逻辑不一致，继续跟下去，发现分歧处在IoEnumerateRegisteredFiltersList函数，原型如下：

{{< highlight cpp >}}
NTSTATUS
    IoEnumerateRegisteredFiltersList(
        IN  PDRIVER_OBJECT  *DriverObjectList,
        IN  ULONG  DriverObjectListSize,
        OUT PULONG  ActualNumberDriverObjects
    );
{{< /highlight >}}
    
其中对DriverObjectList参数的解释，在WDK 7600.16385.1 的文档中是这样：

> “The filter drivers are enumerated in order of decreasing distance from the base file system. The first element (index zero) in the DriverObjectList array represents the filter that is attached farthest from the file system. The second entry is for the next-farthest filter, and so on. The last entry in the array is for the filter that is closest to the base file system. ”

WDK是说，DriverObjectList中的DrvObj是按照顺序排列的，离FSD最远的排最头（第一个元素），反之则排在最末尾。  
可实际调试的结果正好相反，查看了下wrk代码，

{{< highlight cpp >}}
NTSTATUS
    IoEnumerateRegisteredFiltersList(
        IN  PDRIVER_OBJECT  *DriverObjectList,
        IN  ULONG           DriverObjectListSize,
        OUT PULONG          ActualNumberDriverObjects
    )
{
    // ...... 无关代码，略
 
    entry = IopFsNotifyChangeQueueHead.Flink;
 
    while ((numListEntries > 0) && (entry != &IopFsNotifyChangeQueueHead)) {
 
        nPacket = CONTAINING_RECORD( entry, NOTIFICATION_PACKET, ListEntry );
 
        ObReferenceObject(nPacket->DriverObject);
 
        *DriverObjectList = nPacket->DriverObject;
        DriverObjectList++;
 
        entry = entry->Flink;
        numListEntries--;
    }
 
    // ...... 无关代码，略
}
{{< /highlight >}}

可以看出，返回的DriverObjectList中的DrvObj都是从IopFsNotifyChangeQueueHead链表中取出的，而且两个链表中的元素摆放顺序是一致的。

那再来看看IopFsNotifyChangeQueueHead中的元素是按什么顺序摆放的：
{{< highlight cpp >}}
NTSTATUS
    IoRegisterFsRegistrationChange(
        IN PDRIVER_OBJECT DriverObject,
        IN PDRIVER_FS_NOTIFICATION DriverNotificationRoutine
    )
{
    // ...... 无关代码，略
 
    nPacket->DriverObject = DriverObject;
    nPacket->NotificationRoutine = DriverNotificationRoutine;
 
    InsertTailList( &IopFsNotifyChangeQueueHead, &nPacket->ListEntry );
 
    // ...... 无关代码，略
}
{{< /highlight >}}
可以看出，顺序是先来后到，也就是先调用IoRegisterFsRegistrationChange的DrvObj被插在链表的前面。而一般来说，先Attach到FSD的Driver，先调用IoRegisterFsRegistrationChange（一般驱动逻辑如此，你要硬写个驱动挂上去后死活非要在10分钟后Register，我这没辙）。

半路总结下：实际的逻辑和WDK文档上说的不一致。  
为了再次确认，我再瞅瞅我实际调试的环境（Win 7），wrk毕竟是2003版本的。

    1: kd> x nt!IopFsNotifyChangeQueueHead
    83db4770 nt!IopFsNotifyChangeQueueHead =
    
    1: kd> dt _LIST_ENTRY 83db4770
    ntdll!_LIST_ENTRY
    [ 0x869ae690 - 0x8ab5c3a0 ]
    +0×000 Flink : 0x869ae690 _LIST_ENTRY [ 0x8ab5c3a0 - 0x83db4770 ]
    +0×004 Blink : 0x8ab5c3a0 _LIST_ENTRY [ 0x83db4770 - 0x869ae690 ]
    
    1: kd> dd 0x869ae690
    869ae690 8ab5c3a0 83db4770 8c8cc648 843e4bda
    869ae6a0 16160803 d8646641 00000000 00000002
    869ae6b0 00000001 00000006 00000010 00000010
    869ae6c0 00000000 00000000 00000000 00002000
    869ae6d0 00002000 00001000 00000001 000003e9
    869ae6e0 00020066 00000008 00000000 00000000
    869ae6f0 00000000 00000000 00000000 00000000
    869ae700 00000000 00000000 00000000 00000000
    
    1: kd> !drvobj 8c8cc648
    Driver object (8c8cc648) is for:
    \FileSystem\FltMgr
    Driver Extension List: (id , addr)
    Device Object list:
    8cb927a0 8cbc0590 8ca85d00 8ca5b948
    8c8f6eb0 8c8ef660 8c8e9318 8c89d630
    8c89b630
    
    1: kd> dd 0x8ab5c3a0
    8ab5c3a0 83db4770 869ae690 870509d0 8b9ea0aa
    8ab5c3b0 06030203 4b466650 8d5f15f8 8bdc3628
    8ab5c3c0 daf5c87b 5112730d 06070203 416d7441
    8ab5c3d0 870e0940 0000002e 8aa87bd0 c0cf00cf
    8ab5c3e0 0c000002 00690046 0065006c 0061004e
    8ab5c3f0 0065006d 0061004d 00570070 877c0000
    8ab5c400 00260207 6e664d46 8ab97ed8 8ab6b5d0
    8ab5c410 00000dac 00000000 8ab5c418 00000000
    
    1: kd> !drvobj 870509d0
    Driver object (870509d0) is for:
    *** ERROR: Module load completed but symbols could not be loaded for FSpy.sys
    \FileSystem\FSpy
    Driver Extension List: (id , addr)
    Device Object list:
    8cc9b900 90c03e68 90c145d8 851d6188
    851e8ca0 8cac0448
    
    1: kd> !devstack 8cb927a0
    !DevObj !DrvObj !DevExt ObjectName
    90c145d8 \FileSystem\FSpy 90c14690
    > 8cb927a0 \FileSystem\FltMgr 8cb92858
    8cc16020 \FileSystem\Ntfs 8cc160d8

Win7环境里逻辑跟wrk一致。估计又是WDK文档的BUG，可惜WDK Bug Bash已经结束了，要不可以再弄个U盘玩玩（虽然前两个至今还未收到，悲催的UPS）
