---
title: DataBindingLibrary译文-转换器
date: 2018-02-03 21:28:19
categories:
- android
tags:
---
### 转换器
对象转换
当表达式返回一个对象时,将会从自动set,重命名set,自定义set中自动选择set方法,根据set方法的参数类型,强制转换为对应的类型.
在用ObservableMaps存放数据时比较方便,比如:

```
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

这里userMap返回一个对象,这个对象将会根据`setText(CharSequence)`方法进行强制类型转换.但是某些时候直接强制类型转换是不行的,此时开发者需要手动进行转换.

#### 自定义的转换器

下面这种情况:

```
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

此处background需要一个`Drawable`,但是实际上color是一个`integer`对象.当出现这种情况时,需要将int转换为`ColorDrawable`.这个转换是通过`BindConversion`注解的静态方法实现的.

```java
@BindingConversion
public static ColorDrawable convertColorToDrawable(int color) {
   return new ColorDrawable(color);
}
```

注意:转换只发生在set方法中,所以下面混合的类型是不允许的.

```
<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

#### AndroidStudio对databind的支持

android studio支持较多的databind编辑器特性.目前有如下支持:
* 语法高亮
* 语法错误提示
* xml自动完成
* 快速跳转到类声明,快速查看文档

注意:数组和泛型类型,比如Observable类,可能在没有错误时也显示错误.

你可以通过表达式中使用default属性来指定希望在预览窗口中看到的样式.比如下面这个例子中,预览窗口会显示PLACEHOLDER值作为TextView默认的文字:

```
<TextView android:layout_width=”wrap_content”
   android:layout_height=”wrap_content”
   android:text=”@{user.firstName, default=PLACEHOLDER}”/>
```

如果你希望更简单的在设计阶段显示显示你的默认值,也可以使用tools命名空间来指定.详细使用可以参考Designtime Layout Attributes.