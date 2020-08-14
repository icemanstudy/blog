---
title: 'AS打开Flutter项目,原生android工程标红的问题'
date: 2020-08-14 17:54:12
categories:
- Flutter
tags:
---

**现象:**  
使用AS新建或者打开一个Flutter项目后,原生android工程部分,MainActivity中FlutterActivity标红,提示找不到FlutterActivity类.
![](https://raw.githubusercontent.com/icemanstudy/ImageStore/master/20200808171746.png)

#### 实际上,关于在原生工程中进行Flutter关联开发所需要的所有类,都引用失败了.

虽然进行Flutter开发没有问题,main.dart中的代码自动完成也运行正常.但是如果需要同时开发dart和原生,就很不方便了,原生工程里面一片红,代码自动完成完全失效了.为此,特地探究一下报错的原因.

网上的做法千篇一律:

> 删掉android和ios两个目录,然后用"flutter create . "命令重新创建两个原生工程.

> PS:flutter create -a java .可以创建使用java语言的andorid工程(默认是koltin).

### 然而.并没有什么卵用.(AS 3.6.3/4.0.1,flutter 1.20.1 stable)

仔细观察项目,发现一点猫腻.
![](https://raw.githubusercontent.com/icemanstudy/ImageStore/master/20200814173629.png)

为什么这个app**文件夹上没有图标**呢?

没有图标就意味着没有识别出这是一个AS支持的工程,进而意味着不会解析下面的build.gradle,然后就不会加载各种依赖.

**总而言之AS认不出来,那么这就是个普通文件夹.还不如用文本编辑器打开呢,好歹没有红色报错.**

AS打开普通的andorid工程,是靠解析对应文件夹下面的build.gradle和setttings.gradle.然而这个app工程并不在当前根目录下,其父目录android本该作为根目录由AS打开,实际上由于还有flutter,ios相关文件夹,当前AS打开的是其父目录flutterstudy.

那么直接用AS打开android文件夹试试?

编译,运行没有问题.done~

![](https://raw.githubusercontent.com/icemanstudy/ImageStore/master/20200808180444.png)

顺便瞄了眼这个FlutterActivity所在的包,为啥就依赖不到呢.

![](https://raw.githubusercontent.com/icemanstudy/ImageStore/master/20200808180513.png)

看来是一个叫flutter_embedding_debug:1.0.0的依赖.用于原生项目可以加载dart产物.然而由于上述的原因,根本没有加载,导致原生工程啥也做不了.

弄来弄去,只是明白了原因,但问题并没有解决.退而求其次:  
- 做flutter开发时,打开flutter工程.  
- 做原生开发时,打开内部的原生工程.

等待flutter插件的更新吧~
