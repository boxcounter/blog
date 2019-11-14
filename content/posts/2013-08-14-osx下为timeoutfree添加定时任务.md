---
title:  osx下为timeoutfree添加定时任务
date:   2013-08-14
tags            : ["工具"]
category        : "工具"
isCJKLanguage   : true
published: false
---

　　我习惯使用休息提醒软件定时提醒我站起来扭扭腰，目前使用的是“Time Out Free”，挺好用的一款免费软件。

　　但是使用过程中我有一个用着不够爽的地方：  
　　下班回家后或者周末，我基本是躺着或者趴在床上用电脑，不需要再提醒我。但是它没有提供时间段配置，于是我用了定时任务来实现了，如下：

　　下载附件“timeoutfree.sh”，加上执行权限。然后在终端中输入“crontab -e”编辑个人的定时任务，在里面添加如下两行（路径根据情况调整）：

    */20	9-10	*	*	1-5	/Users/boxcounter/timeoutfree.sh start
    */20	18-23	*	*	1-5	/Users/boxcounter/timeoutfree.sh stop
　　然后保存退出，终端应该会输出“crontab: installing new crontab”，说明添加定时任务成功了。

　　说明：  
　　第一行的意思是：工作日每天早上9～11点这两个小时内，每隔20分钟尝试一次启动TimeOutFree。  
　　第二行的意思是：工作日每天晚上18～24点这几个小时内，每隔20分钟尝试一次关闭TimeOutFree。

　　附件：

　　（脚本可能看起来很麻烦，是我随手找了个守护程序的脚本改的。）

