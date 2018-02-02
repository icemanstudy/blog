---
title: python Web问题记录
date: 2018-02-02 10:00:49
categories:
- python
tags:
---
### 1.如何在虚拟主机上安装需要的模块

虚拟主机上默认只有安装python开发环境.如何安装flask,flask-bootstrap这种?

本机上安装,使用pip install xxxx,如pip install flask,

在web配置时,则在requirements.txt中加上需要的模块名,与pip install xxx中的xxx一直,不然可能找不到对应的模块下载.

比如我当前项目中的requirement中只有

Flask==0.10.1

flask-bootstrap

有”=”号的时候,会按指定版本下载,没有”=”号则下载最新版本.

相关提问链接:http://segmentfault.com/q/1010000002653845

### 2.python web中make_server方式和直接运行有什么区别?

这个我还不是很懂.

先上提问链接,以后完善

http://segmentfault.com/q/1010000002652212

### 3.使用pycharm时,如何带参数运行?

使用 pycharm 的 run 功能，只是执行 python hello.py, 并没有添加 runserver 参数。因此脚本没有参数就执行完毕了。

在 run选项的下拉箭头，选择 edit configure选项，然后在 script parameters里添加即可

http://segmentfault.com/q/1010000002653650