---
title: hexo d没反应的问题
date: 2018-02-01 15:34:11
categories:
- 杂技
index_img:
- /img/post/17.jpg
excerpt:
- 吐槽一下hexo异常严格的格式要求.
---
想重新整理一下github上博客.结果死活deploy不上去.
执行hexo d就是没反应.不报错也没输出.

### 少了什么插件?
按网上的说法尝试安装npm install hexo-deploy-git --save.由于事情已经过去了,插件名字可能没写对,反正装上去了也没用.

### 必须使用https的git地址+用户名密码?
虽说教程都是用https,但是deploy节点中配置的也就是一个标准的git remote信息,ssh的方式应该不至于用不了,而且用ssh部署也是有先例的.

最后在v2ex上一篇文章最后一个评论中找到答案...原来是空格问题.
{% asset_img 微信截图_20180201154010.png %}

于是我在每个字段前面加个两个空格,搞定.
{% asset_img 微信截图_20180201154205.png %}

这尼玛,语法要求这么严格!!!