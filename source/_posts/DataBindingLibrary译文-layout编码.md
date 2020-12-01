---
title: DataBindingLibrary译文-layout编码
date: 2018-02-03 18:57:57
categories:
- android
tags:
index_img:
- /img/post/9.jpg
---
### Layout编码介绍
#### Import
你可以在data元素内部使用任意数量的import元素.类似于java中的import,这样可以很方便的导入其他的类.

```
import type="android.view.View"
```

现在你可以在bind表达式中使用View的属性了.

```
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

如果class名称有冲突,你需要给其中一个起个别名

```
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

现在layout文件中,Vista可以用来代表com.example.real.estate.View,而View则还是表示android.view.View.
导入的类可以用于变量或者表达式中.

```
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User&gt;"/>
</data>
```

>注意:AndroidStudio目前还不支持使用import类时变量名称的自动完成.这不影响整个项目的正常编译,不过你可能需要花一些时间在手写完整变量名称上面.

*译者注:截止20170315,android studio2.3中已经可以自动完成变量名了.*

```
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

import类型同样可以用于在表达式中调用静态方法

```
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>

<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

跟java一样,java.lang.*是默认导入的.

#### 变量

data元素中可以放入任意数量的变量.每个变量描述了即将绑定于layout中的一个对象类型.

```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```

变量类型会在编译期间检测,所以如果一个变量实现了Observable或者是一个observable容器,它将会反射为对应的类型.否则将会被忽略.

如果针对不同的情况配置了不同的layout文件(比如横屏和竖屏),变量将会被合并.所以你必须保证这些layout文件之间不会有冲突.

针对layout文件中变量使用到的属性,编译过程中自动生成的绑定类将会对这些属性生成get/set方法,属性的默认值遵循java的定义,引用返回null,int返回0.
如果你自己已经定义了get方法,会以get方法返回的为准.

特别的,一个名为context的变量将会自动生成,以便于在表达式中使用,这个context实际上就是来源于当前view的getContext方法.如果你在layout中手动定义了一个变量名字也叫context,那么这个默认的context会被覆盖.

#### 自定义绑定类的名称

默认情况下,绑定类会根据layout文件名执行一定的规则来生成:首字母大写,移除下划线,并把每个下划线后面的第一个字母大写,最后加上后缀Binding.这个类会被放置在module的databinding包里面(build/generatd/source/apt/debug/xxxxx包名/databinding).
例如一个名为contact_item.xml的layout文件,将会生成一个名为ContactItemBinding的类.如果module包名是com.example.my.app,那么这个类将会被放置在com.example.my.app.databinding下面.

使用data元素的class属性,你可以定义绑定类的名字.比如

```
<data class="ContactItem">
    ...
</data>
```

这样会将原本自动生成的绑定类名修改为ContactItem这个名字.但是放置位置不变.还是在module包名/databinding下面.如果需要放在包名下的别的目录.可以这样,使用.前缀是最简单的.

```
<data class=".ContactItem">
    ...
</data>
```

这样ContactItem类会直接放在包名下面.或者你可以直接指定任意包名:

```
<data class="com.example.ContactItem">
    ...
</data>
```

#### 引入

通过定义application的命名空间和变量的名字,变量可以传递给使用included方式引入的layout文件.

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

这里要求必须在name.xml和contact.xml中都定义了user变量.

>注意:databind框架并不支持在merge标签中直接引入一个layout.

比如下面这种是不支持的:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

#### 表达式语法

**常见的支持语法**

表达式语法很多方面类似于java,下面这些用法是一样的:
数学运算符 + - / * %
字符串拼接 +
逻辑运算 && ||
位运算 & | ^
一元运算符 + - ! ~
移位 >> >>> <<
比较运算 == > < >= <=
类型判断 instanceof
括号 ()
常量 字符串,数字
强制类型转换 cast
方法调用
属性访问
数组元素访问
三元运算符 ?:

举个栗子:

```
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
不支持的运算符
下面这些是java中常用,但是目前表达式中不支持的

this
super
new
Explicit generic invocation(我也不知道这是什么鬼)

合并的判空运算符
使用??运算符,优先选择左边的,如果左边的为null,使用右边的.
```
android:text="@{user.displayName ?? user.lastName}"
```
上面的等价于
```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```
自动避免空指针异常
生成的绑定类会自动避免空指针异常.比如在这个表达式@{user.name}中,如果user是null,user.name将会定义为null.同样如果你使用的是user.age,age是一个`int``类型,将会得到默认值0.

集合
通常的集合:arrays, lists, sparse lists, maps,都可以使用[]来方便的访问其内容.

```
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&gt;"/>
    <variable name="sparse" type="SparseArray&lt;String&gt;"/>
    <variable name="map" type="Map&lt;String, String&gt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>


android:text="@{list[index]}"
android:text="@{sparse[index]}"
android:text="@{map[key]}"
```
字符串常量
layout中使用单引号来来指定value,则可以在内部使用双引号.

```
android:text='@{map["firstName"]}'
```
反过来layout中使用双引号(默认),则内部可以使用单引号或者反引号

```
android:text="@{map[`firstName`]}"
android:text="@{map['firstName']}"
```
资源
你可以像以前一样的语法来使用资源id.

```
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```
FormatString也是可以用的.*(译者注:这里本来还有个plurals资源,不过用得实在太少了.)*

```
android:text=”@{@string/nameFormat(firstName, lastName)}”
```
部分资源需要使用修改过的名称: