---
title: gitee+hexo+smms打造免费个人博客
date: 2020-12-01 16:41:54
categories:
- 闲聊
index_img:
- https://i.loli.net/2020/12/01/ily1SYcpaZsqbHL.png
excerpt:
- 程序员blog白嫖指南
---
 博客,英文名blog,大部分时候,是一个对外分(zhuang)享(bi)的地方.

 初为程序员的时候,也曾在csdn,博客园上面写过一些文章.后期由于上网需要接触了vps,借着服务商的支持,一键安装了wordpress,再买个域名,开始折腾"真正"的个人博客.之后看到掘金,简书之类的新生代博客平台,往往嗤之以鼻:"这不就是文艺青年版csdn么?"

 随着热情慢慢减退,自建vps变成直接使用机场,个人博客也随之关闭,加之名利之心看淡,开始寻找一个真正可以记录一些东西,又不用花太多功夫打理的地方.再说了,互联网的世界,不白嫖一波还能叫程序员?


---

本文记录一下免费博客的组成,最终效果可以参考本博客,如假包换,童叟无欺.

 总体来说,一篇文章从产生到让互联网上的观众看到,有三个步骤:
 **写作>生成>部署**
 
![](https://i.loli.net/2020/12/01/ily1SYcpaZsqbHL.png)

 ## 文章写作
 
###  语言选择

**Markdown**
 
 作为目前程序员届通用的文档标记语言,其特性这里就不多说了.[介绍在此](https://zh.m.wikipedia.org/zh-hans/Markdown).
 
###  编辑器

Markdown编辑器的重要指标依次排序:
1. 实时预览
1. 便捷的工具条
1. 左右分栏显示

符合条件的专用编辑器应该不少,这里推荐一个"一物多用"的工具:有道云笔记.

作为一款免费云笔记,不知从什么时候开始,其居然开始支持md编辑,并且还有非常方便的工具条.

![](https://i.loli.net/2020/12/01/3G5FfDP96sAeW1K.png)

目前在移动端没有办法查看md文档,但是如果仅用来写作,已经完全够用了.

### 图床

图片存放于何处,是每一个博客都绕不过去的话题.

SM.MS在众多收费图床中是一股清流.

![](https://i.loli.net/2020/12/01/u8OHiKC31LTZvop.png)

免费用户5G空间,单图片限制5MB,每分钟最多上传20张.完全够用了.

重点是:**支持API上传**.这个妙用后面再讲.

图片的来源有很多,有本地文件,也有截图,如果用GUI方式上传,还涉及到命名问题,易混淆.那么有没有更方便的上传方式呢?

### 图片上传工具

[PicGo](https://github.com/Molunerfinn/PicGo): 一个用于快速上传图片并获取图片 URL 链接的工具.

![](https://i.loli.net/2020/12/01/rbaFq6nB8JxKtNW.png)

本地文件拖动上传这种小儿科操作就不说了.

微信/QQ等通讯软件使用快捷键截图后,直接点击即可上传,并自动复制md格式图片地址,在实际使用中尤其方便.

![](https://i.loli.net/2020/12/01/1XLv7cAOBaPx9sn.png)

其支持的图床中,要么需要收费,要么需要绑定域名,未读前面所说的SM.MS都不需要.完美.

使用起来也很简单,去图床上申请token,填入picgo即可.

![](https://i.loli.net/2020/12/01/Ztq7RfvehUDGJ1m.png)

## 页面生成

借助各种编辑器,md的本地预览是没有问题的,但是想在网页上展示,需要将其转换成html页面.

本文使用的是[hexo](https://hexo.io/zh-cn/),基于nodejs开发,操作简单,而且有各种主题支持.

![](https://i.loli.net/2020/12/01/GxIt4CRpi2L3rJs.png)

## 博客部署

[GitHub Pages](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/about-github-pages)提供了静态页面托管.只需要在自己空间中建立一个和用户名相同的项目即可.

![](https://i.loli.net/2020/12/01/OCqdwuFYHh5BLGm.png)

但是由于网络原因,部署在GitHub Pages上的页面通常打开速度不是很理想.这里本文使用了国内替代品:[Gitee Pages](https://gitee.com/help/articles/4136).

![](https://i.loli.net/2020/12/01/ETbRP91F32IMDfX.png)

与GitHub Pages相比,唯一的缺点是不能自动发布(做成收费功能了...),需要进入项目的服务页面,手动点击更新.

![](https://i.loli.net/2020/12/01/M6qaFRrDVojPIQl.png)

相对于文章发布的低频率,可以忍受.

## 总结

- Gitee Pages提供部署容器.
- Hexo提供页面生成技术.
- SM.MS+PicGo提供快捷图片上传和使用.
- 有道云笔记提供Markdown编辑器功能.

本博客所使用的主题:[Fluid](https://github.com/fluid-dev/hexo-theme-fluid).

