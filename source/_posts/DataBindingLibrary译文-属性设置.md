---
title: DataBindingLibrary译文-属性设置
date: 2018-02-03 21:19:40
categories:
- android
tags:
---
### 属性设置
当一个被绑定属性变更时,绑定类会自动调用通过表达式和view关联起来的set方法.databind框架通常会自动决定调用哪个方法去设置对应的值.

**自动的set方法**
对应每个属性,databind会尝试寻找这个属性对应的set方法.该方法和命名空间无关,仅仅和属性名本身有关.例如,TextView通过android:text属性指定的表达式,将会寻找名为setText(String)的方法.如果表达式返回值为int,databind将会寻找setText(int)方法.所以当心表达式的返回值,如果必要的话,最好进行一下转换.注意:databind不管view实际上是否有对应的属性.,都会尝试寻找setXXX方法.所以根据这种机制,你可以”创造”属性用于databind.例如support包中的DrawerLayout在xml中没有任何属性,但是在java代码中却有很多set方法.所以你可以使用自动set机制去使用它们.

```
`<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}"/>
```

_译者注:上面的例子中app:scrimColor=”@{@color/scrim}”等同于setScrimColor(getColor(R.color.scrim)),但是直接使用app:scrimColor=”#33ff0000”这样的会编译报错.所以需要使用表达式来对xml中的属性进行赋值,databing才会自动调用对应的set方法_

#### 重命名set方法

有些属性的set方法并不完全和属性名称对应.对于这些方法,需要通过一个叫BindingMethods的注解来进行方法关联.每一个方法都要使用这个注解来封装属性和对应方法名.例如,`android:tint`属性实际上和`setImageTintList`关联了,而并非setTint.
_译者注:参考databind源码:android.databinding.adapters.ImageViewBindingAdapter_

```java
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```

对于开发者而言,并不需要进行这样的重命名.android系统已经实现了这样的转换.

_译者注:为什么有时候是app:有时候是android:,前者自动set方法仅仅是根据属性名自动寻找对应的set方法,是无视namespace前缀的.但是后者由自带的绑定类进行了重命名,会转换为指定的set方法._

#### 自定义的set方法

有些属性需要根据代码逻辑进行自定义绑定.例如并没有一个方法对应android:paddingLeft属性,实际上存在的方法是`setPadding(left, top, right, bottom)`.这里我们可以使用一个静态的绑定方法,使用BindAdapter注解来帮助开发者来定义一个对应该属性需要被调用的方法.

在ViewBindingAdapter中已经内置了一些自定义的set方法.比如paddingLeft:

```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
   view.setPadding(padding,
                   view.getPaddingTop(),
                   view.getPaddingRight(),
                   view.getPaddingBottom());
}
```

BindingAdapter比较适合用于一些拥有自定义属性的情形.例如,你可以通过这种方式在子线程中加载图片.
_当命名冲突时,开发者声明的bind adapter方法会覆盖掉默认的_
你还可以给adapter定义多个参数用于接收:

```java
@BindingAdapter({“bind:imageUrl”, “bind:error”})
public static void loadImage(ImageView view, String url, Drawable error) {
   Picasso.with(view.getContext()).load(url).error(error).into(view);
}
```

```
<ImageView app:imageUrl=”@{venue.imageUrl}”
app:error=”@{@drawable/venueError}”/>
```

如果bimageUrl和error同时设置在这个ImageView上面,并且值类型匹配,上面这个adapter的loadimage方法将会被调用.

上面这种写法默认是需要两个参数同时设置.如果仅设置了其中之一,会在编译阶段报错.或者可以采用下面这种写法:

```java
@BindingAdapter(value = {"imageUrl", "error"}, requireAll = false)
    public static void loadImage(ImageView view, String img, String error) {
        System.out.println("loadImage:" + img + ":" + error);
    }
```

将requireAll的默认值修改为false.就可以在xml中任意配置,未配置的会返回该参数类型的默认值.
自定义的namespace在匹配时会忽略.如上面的例子
你可以直接使用android命名空间.

bind adapter可以在执行方法时获取旧的值,如果你定义的方法参数中有旧的值,那你必须把旧值放在参数列表前面,新值放在参数列表后面,binddata会自动识别是否有旧值:

```java
@BindingAdapter(“android:paddingLeft”) 
public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
	if (oldPadding != newPadding) { 
		view.setPadding(newPadding, view.getPaddingTop(), view.getPaddingRight(), view.getPaddingBottom()); 
		} 
	}
```

有时候参数可能是接口实现类.比如

```java
@BindingAdapter(“android:onLayoutChange”) 
public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue, View.OnLayoutChangeListener newValue) {
	if (Build.VERSION.SDK_INT &gt;= Build.VERSION_CODES.HONEYCOMB) {
		if (oldValue != null) { 
			view.removeOnLayoutChangeListener(oldValue); }
		if (newValue != null) { 
			view.addOnLayoutChangeListener(newValue); } 
	} 
	}
```

译者注:值发生变更依然还是根据表达式中引用的变量发生改变,然后调用notifyPropertyChanged引起.BindAdapter只是定义了xml中key和对应的model属性如何关联起来.

当表达式中监听的方法是某个listener方法之一时,需要将该listener下每个方法进行单独定义.比如View.OnAttachStateChangeListener有两个方法:onViewAttachedToWindow()和onViewDetachedFromWindow(),我们必须创建两个接口来区分和捕获它们.

```java
@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewDetachedFromWindow {
void onViewDetachedFromWindow(View v);
}

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewAttachedToWindow {

void onViewAttachedToWindow(View v);
}
```

因为改变其中一个会影响到另一个(在java代码中,这两个监听是通过同一个方法一起设置到某个view上面),所以我们要定义3个绑定方法,对应只定义一个和同时定义两个的情况.

```java
@BindingAdapter(“android:onViewAttachedToWindow”)
public static void setListener(View view, OnViewAttachedToWindow attached) {
	setListener(view, null, attached);
}

@BindingAdapter(“android:onViewDetachedFromWindow”)
public static void setListener(View view, OnViewDetachedFromWindow detached) {
	setListener(view, detached, null);
}

@BindingAdapter({“android:onViewDetachedFromWindow”, “android:onViewAttachedToWindow”})
public static void setListener(View view, final OnViewDetachedFromWindow detach,final OnViewAttachedToWindow attach) {
	if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
		final OnAttachStateChangeListener newListener;
		if (detach == null && attach == null) {
		newListener = null;
	} else {
		newListener = new OnAttachStateChangeListener() {
			@Override
			public void onViewAttachedToWindow(View v) {
				if (attach != null) {
					attach.onViewAttachedToWindow(v);
				}
			}

			@Override
			public void onViewDetachedFromWindow(View v) {
				if (detach != null) {
				detach.onViewDetachedFromWindow(v);
				}
			}
		 };
	}
	final OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view,newListener, R.id.onAttachStateChangeListener);
	if (oldListener != null) {
		view.removeOnAttachStateChangeListener(oldListener);
	}
	if (newListener != null) {
		view.addOnAttachStateChangeListener(newListener);
	}
}
```

上面这个例子看起来比通常的情况要复杂,这是因为有些listener需要使用add/remove而并非简单的set方法.比如View.OnAttachStateChangeListener.工具类android.databinding.adapters.ListenerUtil可以帮助开发者有效的管理view上面的listener情况.

通过@TargetApi(VERSION_CODES.HONEYCOMB_MR1)注解,databind会自动根据当前运行sdk版本来决定要不要生成对应的代码.