---
title: 在linux上将默认python2换成python3
date: 2018-02-02 10:04:50
categories:
- python
tags:
index_img:
- /img/post/10.jpg
excerpt:
- python3什么时候能够普及呢.
---

>注意：更新python千万不要把老版本的删除！新老版本是可以共存的，很多基本的命令、软件包都要依赖预装的老版本python的，比如yum。

## 更新python：

### 第1步：

更新gcc，因为gcc版本太老会导致新版本python包编译不成功

#yum -y install gcc

系统会自动下载并安装或更新，等它自己结束

安装这四个包…这是安装pip需要的,不然后面pip安装不上.
```
yum install zlib

yum install zlib-devel

yum install openssl -y

yum install openssl-devel -y
```
### 第2步：

下载Python-3.3.0软件包
```
#wget http://python.org/ftp/python/3.3.0/Python-3.3.0.tar.bz2
```

注意：按照上述命令下载的软件包会存放在你当前的工作目录下，wget命令是一个从网络上自动下载文件的自由工具。

说明：命令中的数字就是版本号，你也可以把3.3.0换成你需要的版本，截止至我撰稿时(2013年1月29日)，最新可用版本是3.3.0

### 第3步：

解压已下载的二进制包并编译安装
```
#tar -jxvf Python-3.3.0.tar.bz2 #cd Python-3.3.0

#./configure #make all

#make install

#make clean

#make distclean
```

/usr/local/bin/python3 –V
编译安装完毕以后，可以输入上面一行命令，查看版本

### 第4步：

建立软连接指向到当前系统默认python命令的bin目录让系统使用新版本python
```
#mv /usr/bin/python /usr/bin/python2.6//当前python的版本为2.6所以是python2.6

#ln -s /usr/local/bin/python3.3 /usr/bin/python

#python -V
```

即可查看当前默认python版本 默认的python成功指向3.3.0以后，yum不能正常使用，需要修改yum的配置文件

### 第5步：

修改yum配置文件
```
#vi /usr/bin/yum
```

把文件头部的#!/usr/bin/python改成#!/usr/bin/python2.6 //改为之前的老版本号 保存退出，yum即可正常使用。

如若有其他命令、软件不能正常使用，仿照yum配置文件的修改方法，修改其配置文件即可。 至此，更新完毕。

### 第6步:

安装setuptools和pip

我用的是这两个文件.setuptools-14.0.tar 1.5.5.tar

解压后,找到setup.py用python命令进行安装即可

剩下的就可以用pip安装各种插件了.比如pip install Flask