---
title           : "关于STATUS_INVALID_DEVICE_OBJECT_PARAMETER和STATUS_MOUNT_POINT_NOT_RESOLVED"
date            : 2012-05-20
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

　　很久之前，我写了篇[关于STATUS_INVALID_DEVICE_OBJECT_PARAMETER的blog](http://www.boxcounter.com/technique/2012-05-19-status_invalid_device_object_parameter/)，里面提到使用STATUS_REPARSE的FsFilter（后简称为R-FsFilter）对上层FsFilter造成的恶劣干扰。

　　这两天看Alex Carp的文章，发现事情貌似有转机。具体参考他的[Reparsing to a Different Volume in Win7 and Win8](http://fsfilters.blogspot.com/2012/02/reparsing-to-different-volume-in-win7.html)（需要爬墙）。

　　我总结一下：
　　目前的win8提供了一种新的ECP和相应的数据结构，保证在返回STATUS\_MOUNT\_POINT\_NOT\_RESOLVED的时候开发者可以获得目标FO的FLT\_FILE\_NAME\_INFORMATION和FLT\_INSTANCE。
　　用我那篇blog的原型就是，当驱动A调用FltCreateFile失败后（Alex的文章里只提到了STATUS\_MOUNT\_POINT\_NOT\_RESOLVED，但我猜测对STATUS\_INVALID\_DEVICE\_OBJECT\_PARAMETER应该也是适用的，我还没有动手测试，只是看IopCheckTopDeviceHint的实现做出的猜测），可以从ECP中获取到本次打开操作的目标FO的FLT\_FILE\_NAME\_INFORMATION和FLT\_INSTANCE，然后再次FltCreateFile就可以了。

　　说来简单，但实际上影响挺大的，这相当于要求所有的FsFilter开发者在调用FltCreateFile时都处理这种错误。仅仅是因为目标系统里可能会存在一个R-FsFilter。

　　相信不会有太多的开发者会多写这些代码。
　　而且这个错误对上层FsFilter的开发者来说根本就不是问题。如果在客户机上出现这种问题，完全可以如此解释：“你看，卸载您安装的这个产品（姑且称为T）的驱动后，我公司的产品就能正常工作了。但是只要一安装它，我公司的产品就出问题了，你看你看，另一个公司的产品也不能工作了。这明显是T的问题，跟我公司的产品没有半毛钱关系。”

　　悲催的是R-FsFilter开发者，知道问题有解决方案，但是却不能在自己的代码里解决……。

　　或者，R-FsFilter开发者在开发完之后，再多写一个紧贴着它的上层驱动，专门处理STATUS\_MOUNT\_POINT\_NOT\_RESOLVED/STATUS\_INVALID\_DEVICE\_OBJECT\_PARAMETER？这要求必须使用支持固定加载层次的minifilter框架，并且MS要足够给面子提供两个相邻的Altitude，不过以我之前申请Altitude的经验，貌似有些难度。

　　另外推荐Alex-Cap的几篇关于reparse技术的博文。其实他blog的每篇技术文章都推荐仔细阅读，我几乎备份了他所有的博文……

　　注：由于本人能力有限，行文难免有错，请不吝赐教，感谢。
