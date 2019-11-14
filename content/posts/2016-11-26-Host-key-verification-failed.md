---
title           : "Host key verification failed的另类触发"
date            : 2016-11-26
tags            : ["SSH"]
category        : "研发"
isCJKLanguage   : true
---

最近几天在编写、测试部署脚本，准备配合Jenkins进行自动化的持续集成，因此需要全程静默执行，未想触发了一个SSH key问题，卡了我好几个小时……

# 问题现象

从GitHub上clone过仓库的码农们都知道，用SSH比HTTPs更可靠。但SSH初次连接服务器时，通常会询问是否接受连接，比如这样：

> The authenticity of host 'github.com (192.30.252.1)' can't be established.  
> RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.  
> Are you sure you want to continue connecting (yes/no)?

通常大家手输yes就可以继续clone代码了。而静默执行的脚本通常会在建立连接之前将目标站点保存到`known_hosts`，以跳过询问。

在我的脚本里是这样的：

```
ssh_dir="${HOME}/.ssh"
...
ssh-keyscan -v -t rsa ssh.github.com > ${ssh_dir}/known_hosts
git clone git@github.com:...
...
```

但诡异的是在Jenkins部署过程中并没有生效，还是在建立SSH连接时卡住然后报错「Host key verification failed」。如果打开pty选项，则会如前面那样询问是否接受连接。而在我的本地虚拟机中则完全正常、全程安静地我都不好意思。

# 初步尝试

有怀疑过：

- 是不是SSH目录或其中文件的权限设置不对，导致SSH无法读取？
- 是不是known_hosts没有生效？格式不对？

搜遍资料、试遍方法，几个小时过去了，依然没有进展。眼瞅着就没法轻松过周末了……

# 新思路

于是换了个思路，跳过Jenkins、直接SSH到被部署服务器上，手动执行脚本：

```
chmod u+x ./init.sh
sudo ./init.sh
```

又看到了熟悉的询问，悬着手腕琢磨接下来该怎么办，转念一想「要不yes了看看它自己能不能保存到known_hosts、保存的格式是怎样的」。动手之前先`mv known_hosts known_hosts.old`待会好diff。

# 峰回路转

输入yes，峰回路转 —— 并没有生成known_host。

又陷入思考「难道权限不够？不对啊，我已经sudo了」，一想到sudo，顿时一个激灵 —— 当前是root帐号！赶紧`sudo ls -la /root/.ssh/`，果然躺着一个known_hosts文件。

到此，我才弄明白问题的原因：

1. 因为sudo，脚本是以root帐号身份执行，会去`/root/.ssh/known_hosts`找known_hosts；
2. 脚本的ssh-keyscan将ssh.github.com保存进了`~/.ssh/known_hosts`，被部署服务器使用的是非root帐号；

SSH就没找到ssh-keyscan的结果，所以询问。而之所以在本地虚拟机里一切正常是因为我偷懒用的是root帐号。

再一细想，我真是糊涂，怎么会整个脚本都sudo呢，这样岂不是git clone等等命令都是以root用户身份执行了吗……

# 解决

找到了问题症结，解决也就简单了，花了十分钟把需要权限的命令都加上sudo，`echo xxx > $file`都改成`echo xxx | sudo tee $file`，去掉Jenkins构建步骤里的sudo 。

Jenkins部署成功，欢度周末～
