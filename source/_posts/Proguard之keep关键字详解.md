---
title: Proguard之keep关键字详解
date: 2018-02-03 18:05:56
categories:
- android
tags:
index_img:
- /img/post/19.jpg
excerpt:
- Proguard其实做的事情比你想象的要多.
---
### 原博客的图片丢失了,所以这是精简版,直接上结论.

项目很简单,只有两个文件:一个activity+一个普通类.

ProguardTest

Activity

build.gradle.标准配置.使用sdk里面默认+自己的空白配置文件

先直接混淆一下看看.

Activity肯定是不混淆的.oncreate方法由于是override也是不混淆的.

而ProguardTest类的类名,成员名(包括方法名和变量名)都被混淆了.包括内部类也是一样.

还有一点:类中未使用的方法setName(),直接被移除掉了.

---

proguard keep选项的结构:({}内容为必须,[]内容为选用)

>{关键字}[类的修饰符]{在哪个类}{keep哪些成员}

下面一条一条试验各个keep关键字的效果.

#### -keep class com.iceman.proguard.ProguardTest{*;}

先套用上面的固定格式解读一下:
关键字–keep

修饰符-无 eg:public

在哪个类–ProguardTest类

keep哪些成员-全部

这个基本没啥好说的.将这个类的名字和成员全部保留起来.未使用的方法setName也保留起来.

#### -keepnames class com.iceman.proguard.ProguardTest{*;}

keepnames
区别在哪里?setName方法不见了.

根据官方的解释keepnames = keep+allowshrinking.即没有用的的类/成员会去除掉.

#### -keepclassmembers class com.iceman.proguard.ProguardTest{
*;}

只保留成员名字.类本身的名字被混淆.

#### -keepclassmembernames class com.iceman.proguard.ProguardTest{
*;}
keepclassmembernames = keepclassmembers+allowshrinking
所以此时setName方法会被去除掉.就不截图了.

下面是重点:keepclasseswithmembers,用两条proguard配置做对比:
-keepclasseswithmembers class com.iceman.proguard.ProguardTest{
        void method1();}
-keepclasseswithmembers class com.iceman.proguard.ProguardTest{
void adfadfaf();}


看来关键在于后面的成员定义这里,

前者找到method1方法了,然后类名和method1这个方法名被keep了.其他混淆.

后者没有找到这个方法adfadfaf方法,所以整个类和成员都被混淆了.

还有加上names的那个.区别参考前面的.names意为去除未使用的.

---
总结keep关键字中几个单词的作用:

names:加上names即允许shrinking.未使用的类/成员会直接去除.不再keep了.

members:仅仅keep成员,对类自身的名字不再keep.

withMembers:当指定的成员存在时,keep类名和对应的成员名.否则全部混淆.

类名定义后面的{;}:大多数情况下为{;}代表keep范围是所有成员,但是如果不写大括号及里面内容的话,keep是不会对成员生效的.仅仅是在某些关键字情况下对类名做keep.
实战示例:

项目中采用了实体类来对应接口返回的json数据.由于使用Gson包做映射,这些实体类的成员变量都是不能混淆的.

通常采用的方式是:定义一个Unproguard接口.凡是需要保留的类,都继承这个接口.
```
-keepclassmembers class * implements rst.framework.interfaces.UnProguard{
*;
}
```
可是实际运行后,发现还是统统被混淆了.经过查询原因,发现UnProguard这个类本身也是需要保留的.

于是加上一条-keep class rst.framework.interfaces.UnProguard后解决.

混淆keep关键字解析到此结束.更详细的使用文档请参考官网