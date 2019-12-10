---
title: android Q适配笔记
date: 2019-07-06 18:12:36
categories: android
tags:
---

### 关于非SDK接口管控

**首先要说明的是,仅在targetSdkVersion调整为29(android Q)的情况下才需要考虑这一点.
如果应用的targetSdkVersion还停留在26(android O)甚至更低,在这一方面可以不用做任何变更.**

官方对于非sdk接口管控是通过黑名单/灰名单列表来管控的.
targetSdkVersion被看做是应用的一项"声明",代表应用已经针对指定版本的平台做了适配工作,承诺会遵守对应平台的新特性(隐私方面,安全方面,等等).


**兼容**-无论如何,android高版本需要支持面向低版本开发的应用  
**适配**-开发者已知高版本特性,并对其做了针对性处理


> blacklist:在targetSdkVersion对应的平台上,该方法不允许调用  
> greylist-max-p:若targetSdkVersion<=P,警告.否则P以上的设备不允许调用.  
> greylist-max-o:若targetSdkVersion<=O,警告.否则O以上的设备不允许调用.  
> greylist:若targetSdkVersion>=P,警告,否则正常调用.

检查方式有三种,建议结合使用:

1. 在android Q设备上运行程序,以Accessing hidden为关键字在log中筛选.
2. 在StrictMode中打开detectNonSdkApiUsage.
3. 使用官方工具veridex扫描apk,有误差,需要以实际logcat为准.

**改造方式:**

部分方法是为了将高版本上的优化措施应用到低版本上,因此调用时可以判断sdk版本,高版本跳过.  
另外大多数方法在调用时都已经加上了try catch,一般保证代码执行流程无影响即可.

### 关于存储沙箱
一般涉及到此类场景,都是保存图片到相册.只需针对处理即可.  
或者直接使用requestLegacyExternalStorage=true来兼容.

### 关于compileSdkVersion升级带来的androidX迁移

从AS中的建议来说,要求是compileSdkVersion>=targetSdkVersion.因此将targetSdkVersion升级后,对应的compileSdkVersion和support库也需要升级.

而从android 29开始,andoridX代替了support库.很大一部分工作量在于andoridX迁移.

在项目根目录的gradle.properties中增加两行代码,然后使用AS自动迁移.

**android.useAndroidX=true**

**android.enableJetifier=true**
//可以将library中的support依赖自动转换为androidX依赖.不论是远程引用还是本地library,不论是java代码还是xml中的控件.

另外附上需要手动替换的部分列表:

> import android.support.annotation.NonNull  
> import androidx.annotation.NonNull
> 
>
> import android.support.annotation.Nullable  
> import androidx.annotation.Nullable
> 
> import android.support.v7.widget  
> import androidx.appcompat.widget
> 
> import androidx.appcompat.widget.CardView  
> import androidx.cardview.widget.CardView
> 
> android.support.v7.appcompat.R  
> androidx.appcompat.R
> 
> import android.support.annotation  
> import androidx.annotation
> 
> import androidx.appcompat.widget.RecyclerView  
> import androidx.recyclerview.widget.RecyclerView
> 
> import androidx.core.app.Fragment  
> import androidx.fragment.app.Fragment
> 
> import androidx.appcompat.widget.LinearLayoutManager  
> import androidx.recyclerview.widget.LinearLayoutManager
> 
> import androidx.core.view.PagerAdapter  
> import androidx.viewpager.widget.PagerAdapter
> 
> import androidx.core.view.ViewPager  
> import androidx.viewpager.widget.ViewPager
> 
> import androidx.core.app.DialogFragment  
> import androidx.fragment.app.DialogFragment
> 
> import androidx.appcompat.widget.SimpleItemAnimator  
> import androidx.recyclerview.widget.SimpleItemAnimator
> 
> import androidx.appcompat.widget.GridLayoutManager  
> import androidx.recyclerview.widget.GridLayoutManager
> 
> import android.support.v7.app.AppCompatActivity  
> import androidx.appcompat.app.AppCompatActivity
> 
> import android.support.v7.app.AlertDialog  
> import androidx.appcompat.app.AlertDialog
> 
> import androidx.core.content.LocalBroadcastManager  
> import androidx.localbroadcastmanager.content.LocalBroadcastManager
> 
> import android.support.design.widget.AppBarLayout  
> import com.google.android.material.appbar.AppBarLayout
> 
> import androidx.core.widget.SwipeRefreshLayout  
> import androidx.swiperefreshlayout.widget.SwipeRefreshLayout
> 
> import androidx.appcompat.widget.OrientationHelper  
> import androidx.recyclerview.widget.OrientationHelper
> 
> <android.support.v7.widget.AppCompatImageView  
> <androidx.appcompat.widget.AppCompatImageView
> 
> <androidx.core.view.ViewPager  
> <androidx.viewpager.widget.ViewPager
> 
> <android.support.v7.widget.RecyclerView  
> <androidx.recyclerview.widget.RecyclerView
> 
> </android.support.v7.widget.RecyclerView>  
> </androidx.recyclerview.widget.RecyclerView>
> 
> <android.support.v7.widget.SwitchCompat  
> <androidx.appcompat.widget.SwitchCompat

