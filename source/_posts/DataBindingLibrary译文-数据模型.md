---
title: DataBindingLibrary译文-数据模型
date: 2018-02-03 21:11:20
categories:
- android
tags:
index_img:
- /img/post/14.jpg
---
### 数据模型
任意POJO(简单java对象)都可以用于数据绑定.但是默认情况下,修改一个POJO的属性,并不会引发UI更新.databind真正的威力在于可以给予数据模型在数据发生变化时请求UI更新的能力.这里有3中不同的数据变更通知机制.
Observable objects, observable fields, 和observable collections.

**Observable objects**
实现了Observable接口的类实例会允许bind过程中给它添加一个listener,用来监听这个实例属性的变化.

Observable接口定义了添加和移除listener的机制,但是开发者关心的只是数据变化发起的通知.为了方便开发者,基类BaseObservable被创建,用来自动完成listener添加机制.但是实现方依然有责任在数据发生变化时,发出通知.这里可以通过添加一个Bindable的注解在get方法上,并在set方法中来发出通知.
```java
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```

*译者注,不写get/set方法,直接把注解加载成员变量上面也是可以的.*
编译过程中,会在module package中生成BR类,Bindable注解在这个过程中会在BR类中生成一个变量.
如果您的数据类无法修改，则可以使用 PropertyChangeRegistry 来保存和通知改变事件。_(此处不太懂)_

#### ObservableFields

直接继承BaseObservable类可能有点麻烦.如果你的类只有一小部分属性需要变化并通知,你可以使用ObervableField及其相关类:ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, and ObservableParcelable.
ObervableFields是一个自身继承BaseObservable的对象,内部封装了一个基本对象,访问时并不会涉及封包解包操作.使用时,在数据对象中创建一个public final的属性即可:

```java
private static class User {
   public final ObservableField&lt;String&gt; firstName =
       new ObservableField&lt;&gt;();
   public final ObservableField&lt;String&gt; lastName =
       new ObservableField&lt;&gt;();
   public final ObservableInt age = new ObservableInt();
}
```

这样,操作属性的时候,使用get/set方法即可.

```java
user.firstName.set("Google");
int age = user.age.get();
```

#### Observable Collections

有的应用中可能需要更加灵活的数据结构来存储数据.ObservableCollections支持使用key或者下标来访问这些数据.
当key为对象时,比如String,可以使用ObservableArrayMap:

```java
ObservableArrayMap&lt;String, Object&gt; user = new ObservableArrayMap&lt;&gt;();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);
```

然后在layout中可以通过key来访问数据

```
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>

<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

当key为int时.可以用ObservableArrayList,然后通过下标来访问数据

```java
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```

```
<data>
<import type=”android.databinding.ObservableList”/>
<import type=”com.example.my.app.Fields”/>
<variable name=”user” type=”ObservableList<Object>”/>
</data>

<TextView
android:text=’@{user[Fields.LAST_NAME]}’
android:layout_width=”wrap_content”
android:layout_height=”wrap_content”/>
<TextView
android:text=’@{String.valueOf(1 + (Integer)user[Fields.AGE])}’
android:layout_width=”wrap_content”
android:layout_height=”wrap_content”/>
```