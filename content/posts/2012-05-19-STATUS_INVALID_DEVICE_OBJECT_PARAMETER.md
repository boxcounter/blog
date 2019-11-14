---
title           : "有关STATUS_INVALID_DEVICE_OBJECT_PARAMETER"
date            : 2012-05-19
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

这两天遇到一个问题，记录在这里。
我有两个FS过滤驱动，这里称它们为 A 和 B，A 在 B 之上，其中 B 是个重定向驱动。下面是问题描述：
A 调用 FltCreateFile 打开 C: 中的某个文件，B收到这个请求后，指定这个请求重定向到 D:，结果 A收到了错误码 STATUS_INVALID_DEVICE_OBJECT_PARAMETER。 挂上 WRK 调试，发现整个调用栈是这样：

```
kd> kv
ChildEBP RetAddr Args to Child
b8d9b330 808ee689 b8d9b3fc 846639c8 00000000 nt!IopCheckTopDeviceHint+0x56
b8d9b424 809241d1 84b2d2c8 00000000 84747a18 nt!IopParseDevice+0x417
b8d9b4a4 80920538 00000000 b8d9b4e4 00000240 nt!ObpLookupObjectName+0x5a7
b8d9b4f8 808e1119 00000000 00000000 d9b5b800 nt!ObOpenObjectByName+0xe8
b8d9b574 808e3242 b8d9b710 00000000 b8d9b6f8 nt!IopCreateFile+0x447
b8d9b5bc f732f6ae b8d9b710 00000000 b8d9b6f8 nt!IoCreateFileSpecifyDeviceObjectHint+0x50
b8d9b668 f732f828 84684ad8 84894888 b8d9b710 fltMgr!FltCreateFileEx+0x114
b8d9b6ac b8a3a506 84684ad8 84894888 b8d9b710 fltMgr!FltCreateFile+0x36
b8d9b71c b8a2fb84 84894888 b8d9b784 00000000 A!OpenObject+0xf6
b8d9b758 b8a30151 84894888 b8d9b784 00000001 A!ObjectExists+0xd4
b8d9b78c b8a30f8c 84857f14 8556abc8 b8d9b7ff A!ParentExistsInOriginalPath+0x101
b8d9b87c b8a31a40 84857f14 8556abc8 b8d9b8cb A!PreCreateOfPuppet+0x32c
b8d9b908 b8a4cb48 84857f14 b8d9b92c 00000000 A!UrPreCreate+0x270
b8d9b934 f7337465 84857f14 b8d9b980 b8d9b99c A!FltPreOperationCallback+0x158
b8d9b960 f731d4e8 00000009 00000000 b8d9b99c fltMgr!FltvPreOperation+0x3f
b8d9b9c0 f731ef48 00d9ba04 84857eb8 86200fd8 fltMgr!FltpPerformPreCallbacks+0x2d4
b8d9b9d4 f732d0ad b8d9ba04 f732b540 00000000 fltMgr!FltpPassThroughInternal+0x32
b8d9b9ec f732d59d b8d9ba04 00000000 80a45be4 fltMgr!FltpCreateInternal+0x63
b8d9ba20 809ab1d2 846605c8 86200e48 8490c340 fltMgr!FltpCreate+0x229
b8d9ba50 8081f881 808eeca7 b8d9bb44 808eeca7 nt!IovCallDriver+0x110
```

STATUS_INVALID_DEVICE_OBJECT_PARAMETER 就是由 IopCheckTopDeviceHint 返回的。

继续分析，得到下面的结果：

B 在收到 A 下发的请求时，经过处理，决定重定向这个请求，于是重新设置 FO->FileName 到 'D:\xxxx'，并返回了 STATUS_REPARSE。这个请求回到 ObMgr 后，ObMgr 按照新的路径进行解析，但是在解析的过程中发现新的路径所在的卷的 DO 和指定的 DOHint 不是同一个设备栈的（调用 IopCheckTopDeviceHint 实现），于是就返回上面的错误码。

OK，问题的原因总体描述完毕，继续来描述细节：
关于 DOHint，

FltCreateFile 在我的虚拟机中是调用 FltCreateFileEx 来完成的，而后者又是由 IoCreateFileSpecifyDeviceObjectHint 来实现，来看看 FltCreateFile 的函数原型：

{{< highlight c "linenos=table,hl_lines=4" >}}
NTSTATUS
FltCreateFile(
__in PFLT_FILTER Filter,
__in_opt PFLT_INSTANCE Instance,
__out PHANDLE FileHandle,
__in ACCESS_MASK DesiredAccess,
__in POBJECT_ATTRIBUTES ObjectAttributes,
__out PIO_STATUS_BLOCK IoStatusBlock,
__in_opt PLARGE_INTEGER AllocationSize,
__in ULONG FileAttributes,
__in ULONG ShareAccess,
__in ULONG CreateDisposition,
__in ULONG CreateOptions,
__in_opt PVOID EaBuffer,
__in ULONG EaLength,
__in ULONG Flags
);
{{< /highlight >}}

注意参数 Instance，类型为 PFLT_INSTANCE，看下这个结构：
```
kd> dt _FLT_INSTANCE
fltMgr!_FLT_INSTANCE
+0x000 Base : _FLT_OBJECT
+0x014 OperationRundownRef : Ptr32 _EX_RUNDOWN_REF_CACHE_AWARE
+0x018 Volume : Ptr32 _FLT_VOLUME        <<<---
+0x01c Filter : Ptr32 _FLT_FILTER
+0x020 Flags : _FLT_INSTANCE_FLAGS
+0x024 Altitude : _UNICODE_STRING
+0x02c Name : _UNICODE_STRING
+0x034 FilterLink : _LIST_ENTRY
+0x03c ContextLock : _ERESOURCE
+0x074 Context : Ptr32 _CONTEXT_NODE
+0x078 TrackCompletionNodes : Ptr32 _TRACK_COMPLETION_NODES
+0x07c CallbackNodes : [50] Ptr32 _CALLBACK_NODE
```

注意其中 0x18 偏移处的 Volume，类型为 _FLT_VOLUME，看下这个结构：
```
kd> dt fltMgr!_FLT_VOLUME
+0x000 Base : _FLT_OBJECT
+0x014 Flags : _FLT_VOLUME_FLAGS
+0x018 FileSystemType : _FLT_FILESYSTEM_TYPE
+0x01c DeviceObject : Ptr32 _DEVICE_OBJECT        <<<---
+0x020 DiskDeviceObject : Ptr32 _DEVICE_OBJECT
+0x024 VolumeInNextFrame : Ptr32 _FLT_VOLUME
+0x028 Frame : Ptr32 _FLTP_FRAME
+0x02c DeviceName : _UNICODE_STRING
+0x034 GuidName : _UNICODE_STRING
+0x03c CDODeviceName : _UNICODE_STRING
+0x044 CDODriverName : _UNICODE_STRING
+0x04c InstanceList : _FLT_RESOURCE_LIST_HEAD
+0x090 Callbacks : _CALLBACK_CTRL
+0x220 ContextLock : _ERESOURCE
+0x258 VolumeContexts : _CONTEXT_LIST_CTRL
+0x25c StreamListCtrls : _FLT_RESOURCE_LIST_HEAD
+0x2a0 NameCacheCtrl : _NAME_CACHE_VOLUME_CTRL
+0x2d0 MountNotifyLock : _ERESOURCE
+0x308 HiddenOperationState : [4] _FLTP_TRACK_WHEN_TO_HIDE
```

注意其中 0x1c 偏移处的 DeviceObject，这个就是传递给 IoCreateFileSpecifyDeviceObjectHint 的 DOHint，来看下这个函数的原型：
{{< highlight c "linenos=table,hl_lines=17" >}}
NTSTATUS
IoCreateFileSpecifyDeviceObjectHint(
__out PHANDLE FileHandle,
__in ACCESS_MASK DesiredAccess,
__in POBJECT_ATTRIBUTES ObjectAttributes,
__out PIO_STATUS_BLOCK IoStatusBlock,
__in_opt PLARGE_INTEGER AllocationSize,
__in ULONG FileAttributes,
__in ULONG ShareAccess,
__in ULONG Disposition,
__in ULONG CreateOptions,
__in_opt PVOID EaBuffer,
__in ULONG EaLength,
__in CREATE_FILE_TYPE CreateFileType,
__in_opt PVOID InternalParameters,
__in ULONG Options,
__in PVOID DeviceObject
);
{{< /highlight >}}

上面的 DeviceObject 就是作为最后一个参数传入的。

OK，回到问题，A 调用 FltCreateFile 是要打开 c: 中的某个文件，也就是说传入的Instance是属于 C: 的，再说细一点，就是将代表 C: 的一个 DO 作为 DOHint。而经过 B 的重定向后，新的路径是在 D: 中，IopCheckTopDeviceHint 拿代表 C: 的DOHint，和新解析得到的 D: 的 DO 进行了检查，于是就返回了 STATUS_INVALID_DEVICE_OBJECT_PARAMETER。

到这里问题原因解释完毕了，那么，怎么解决呢？ 答案是：没有解决方案。下面是 MS 文件系统研发人员 Alexandru Carp 的回复：

> We are aware of the problem and are working on this. The intention is to make it available to all downlevel supported OSes (even old ones like Win7 :P ) but we'll see how that goes.”

于是，我试图尝试绕过这个问题，以下是我的两个尝试：
1. 壮士断腕：调整 B 的逻辑，不要跨卷重定向。
2. 依稀记得，可以把某个卷 mount 到另一卷，模拟一个普通的目录来处理，那么是不是可以把重定向的目标做成一个单独的卷，就像 TrueCrypt。然后把这个卷 mount 在 C:，这样是不是就不存在跨卷的问题了。
顺着这个思路去查资料，发现这个技巧需要 NTFS 文件系统的支持。有种不好的预感~ 莫不是这种技巧也基于 reparse 技术？ProcMon 了一下，果然是~ 残念了~

难道只能等 MS 的补丁？

（注：此博文是从[转载文章](http://www.longene.org/forum/viewtopic.php?f=8&t=4007)中恢复的，我已经记不清具体是什么时间写的了。感谢转载的welfear。）
