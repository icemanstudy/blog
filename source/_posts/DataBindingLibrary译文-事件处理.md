---
title: DataBindingLibrary译文-事件处理
date: 2018-02-03 18:48:57
categories:
- android
tags:
---
**事件处理**
databind机制允许你使用表达式来处理来自于view的事件,比如onclick.
事件的名称和实际对应的listener方法名有关,有时间会有一些小变更.
例如onLongClickListener有一个方法onLongClick,所以对应的事件为android:onLongClick.这里有两种方式去处理一个事件:
方法引用:通过表达式,你可以将其指向与之参数匹配的listener方法.当一个表达式被确认指向某个方法的时候,databind框架会在一个listener中将方法引用和对应的data对象绑定起来,然后将这个listener设置给对应的view.如果表达式设置为null,databind框架将不会创建listener,并且使用空的listener来代替.
*译者注:这可能影响到某些view的按下效果*
listener绑定:你可以使用lambda表达式来进行事件分发,databind框架将总是会创建listener并设置给view,当事件发生的时候,listener会使用这个lambda表达式.

**方法引用**
事件可以直接指向某个方法,类似于android:onclick可以定义为activity的一个方法.
但是与view的onclick方法相比最主要的优势是,databind中这种表达式会在编译期间进行处理,所以如果对应的方法不存在或者参数不匹配,会导致编译过程中报错.

>方法引用和监听器绑定最主要的区别在于,实际的监听器实现是在data模型被绑定的时候,而并非是事件被触发的时候,如果你希望当事件发生的时候就能执行对应的代码,你应该使用listener绑定

下面是一个普通的bind表达式示例,使用方法名称当做value值即可.假如你的data模型有这样一个方法:

```java
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```


然后绑定表达式可以这样指派某个view的点击事件:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

**注意:表达式中定义的方法的参数必须保持和实际的listener中方法参数完全一样.**

**listener绑定**

listener绑定用来设置当事件发生时执行某些方法.跟方法引用有点像,但是它允许你使用更加随意的bind表达式.这个特性需要android gradle插件2.0版本以上.

在方法引用中,方法参数必须和listener的方法参数完全一样.但是在listener绑定中,你只需要保证返回值和listener方法一样即可(除非是void).例如,你可以定义这样一个类:

```java
public class Presenter {
    public void onSaveClick(Task task){}
}
```

然后你可以这样绑定一个点击事件

```
<?xml version="1.0" encoding="utf-8"?>
  <layout xmlns:android="http://schemas.android.com/apk/res/android">
      <data>
          <variable name="task" type="com.android.example.Task" />
          <variable name="presenter" type="com.android.example.Presenter" />
      </data>
      <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
          <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
          android:onClick="@{() -> presenter.onSaveClick(task)}" />
      </LinearLayout>
  </layout>
```

listener被lambda表达式代替了,当你在表达式中使用这样的callback型语句时,databind框架会自动帮你生成对应的listener并且注册到事件上,当事件被触发时,对应的表达式将被执行.当表达式被执行的时候是线程安全的.

注意上面的例子,我们并没有像onclick(View)这样为onSaveClick方法定义一个View参数.listener绑定提供了两个选择用于参数定义:你可以忽略所有参数,或者列出所有参数.如果你列出了所有参数,你可以在表达式中使用它.比如上面的表达式可以写成这样:

```
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

或者如果你想在表达式中使用这个参数,你可以这么写

```
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

你可以使用不止一个参数的lambda表达式

```java
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```
```
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

如果事件对应方法返回类型不是void,你的表达式必须返回跟它一样的类型.例如,如果你想监听长按事件.你的表达式必须返回boolean类型的结果.

```java
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
```
```
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

如果表达式遇到了空的实现,databind会返回默认的java基本类型对应的数据.引用会返回null,int会返回0,boolean会返回false.
如果你使用了条件判断,你可以使用void作为一个元素.

**避免使用复杂的listener**

listener表达式具有很强大的功能,可以让你的代码变得非常易读.但是从另一方面来说,复杂的表达式可能会让你的layout文件变得可读性和可维护性较差.你应该尽可能简单的将数据从UI传递到定义的方法中来,然后在方法中再去实现复杂的业务逻辑.
为了防止冲突,有些点击事件需要除了android:onClick以外的名字来定义,比如下面这些方法

Class           Listener                                           Setter Attribute
SearchView      setOnSearchClickListener(View.OnClickListener)     android:onSearchClick
ZoomControls    setOnZoomInClickListener(View.OnClickListener)     android:onZoomIn
ZoomControls    setOnZoomOutClickListener(View.OnClickListener)    android:onZoomOut