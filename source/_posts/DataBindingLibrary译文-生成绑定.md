---
title: DataBindingLibrary译文-生成绑定
date: 2018-02-03 21:16:27
categories:
- android
tags:
---
### 生成绑定
自动生成的绑定类将layout中定义的变量和view关联起来.根据前面的讨论,绑定类的名字和所在的包名是可以定制的.共同点是:所有的绑定类都继承于ViewDataBinding.

#### 生成
绑定关系应该在解析完布局之后马上创建,防止后续的代码影响到布局结构.这里有几种方法可以对一个layout建立绑定关系,最常用的是使用绑定类的静态方法.inflate方法可以同时完成view层级解析和绑定关系建立.(译者注:相对于反复findviewbyid,这种一次解析完成映射可以节省不少时间)
下面是两个简单的绑定示例,

```java
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater);
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater, viewGroup, false);
```

如果你的view已经inflate出来了.仅仅需要绑定.可以这样做:

```java
MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
```

有时候你可能想写一些工具类方法,即并不知道需要绑定类的类型,你可以使用DataBindingUtils:

```java
ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
    parent, attachToParent);
ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
```

#### 组件id

针对layout中指定了id的view组件,会对应生成public final的变量.绑定过程解析一次不举文件,找出带有id的组件.这个比多次findViewById要快很多.例如:

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```

会在绑定类中生成这两个变量

```java
public final TextView firstName;
public final TextView lastName;
```

不指定id也可以使用databind,但是如果后期需要访问这个view的话,还是需要指定id的.

#### 变量

每个变量都会生成get/set方法.

```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```

上面的layout会在bind类中生成以下方法

```java
public abstract com.example.User getUser();
public abstract void setUser(com.example.User user);
public abstract Drawable getImage();
public abstract void setImage(Drawable image);
public abstract String getNote();
public abstract void setNote(String note);
```

#### ViewStubs

ViewStubs和普通view有点不同.它们一开始是不可见的,而且并不会解析到view层级中.只有需要显示的时候,才会替换原有的布局,并显示出自身.
因为ViewStub最终会从view层级中移除,那么绑定类中对应的属性应该也可以及时回收.
但是因为view是final类型的,所以用ViewStubProxy来取代原本的ViewStub,让开发者可以访问这个ViewStub,并且当对应的view显示时,可以访问到那个view.
当解析另外一个布局文件的时候,绑定对象也应该和新的布局关联起来.因此,ViewStubProxy需要监听ViewStub的 OnInflateListener回调接口来建立绑定关系.开发者可以在ViewStubProxy上设置一个OnInflateListener ,当绑定建立的时候,开发者可以收到回调函数.
_(译者注:ViewStub用得实在不多,此处没有试验)_

#### 绑定进阶

**动态数据模型**
有时候,并不能知道具体绑定的数据模型.比如在使用RecyclerView.Adapter这样的适配器时,只有在onBindViewHolder中才能知道对应的数据模型.
这种情况下,假设adapter的layout中都有一个item变量.这样你可以使用setVarible方法来将数据模型和item绑定在一起.

```java
public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}
```
**立即刷新**
当数据模型变化时,绑定关系会在下一个周期中刷新页面,这可能会有一点延迟.如果想要强制刷新,可以使用executePendingBindings)方法.

**后台线程**
只要数据模型不是一个集合,你就可以在任意子线程中进行修改.databind会通过保存到本地来帮你解决同步问题.