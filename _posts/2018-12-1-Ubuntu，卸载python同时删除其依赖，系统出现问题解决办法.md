---
layout: post
title: "Ubuntu下卸载自带python及其依赖，出现的问题"
data: 2018-12-1 13:20:00 +0800
author: "YY"
tags:
    - Bug之路
---

自己给自己挖了个坑，把Ubuntu自带的Python2和Python3及其依赖给删除了，导致系统出现问题。经过一番折腾最终解决。方法如下：

**如果终端还可以打开：**

- 把你自己安装的python版本给卸载了，sudo apt-get remove --auto-remove python3/python2

- 卸载完成直接输入：sudo apt-get install ubuntu-desktop，然后重启问题解决。

**如果终端打不开：**

- 终端都无法打开的话，我推荐一种方法，在Windows桌面使用Xshell（puTTY）链接到你的ubuntu，在Xshell中输入上方1，2方法的命令即可。

