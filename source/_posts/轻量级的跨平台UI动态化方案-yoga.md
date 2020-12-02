---
title: 轻量级的跨平台UI动态化方案-yoga
date: 2020-01-06 17:52:44
categories:
- android
tags:
- 跨平台
index_img:
- /img/post/5.jpg
excerpt:
- UI跨平台渲染,满足90%的企业需要.
---
 时间已到2020年,作为有google官方研发,闲鱼背书的方案,Flutter是越来越火了.

 然而考虑到apple store谜一样的审核策略,有部分比较稳妥的企业,并不倾向于使用这种完全从底层实现的跨平台方案.

 那么,有没有比较轻量级的跨平台方案,只将描述语言统一,由各自客户端解析成原生样式展示呢.这便是facebook推出的UI动态化方案--**yoga**.

### yoga简介

yoga的官方网站:https://yogalayout.com/

github地址:https://github.com/facebook/yoga

yogalayout支持的属性与css中的弹性盒子样式对应,https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout

### 现有的yoga使用示例

#### 某不知名github主的demo
https://github.com/koudle/AndYogaSample

clone下来试用之,发现其只是非常粗浅的使用:

{% asset_img image1.png %}

**每一个YogaNode都要指定宽高,指定放置方式,最后一起计算得到最终的每个node的坐标,还不支持dp...**


#### QQ音乐团队发布的yoga使用示例
https://cloud.tencent.com/developer/article/1006148

其中YogaNode的创建和布局方式来源于上一个,但是最终居然还是创建出原生控件然后逐个设置X,Y.这有什么意义...

{% asset_img image2.png %}

**我都知道了每个YogaNode的尺寸,位置,还需要用yoga?直接给控件设置上去不好么???**

不客气的说,前面两个都是屎.

#### yoga官方的demo
https://github.com/facebook/yoga/tree/master/android

原来还有针对android的自定义控件YogaLayout.类似google的FlexBoxLayout,通过自定义控件来支持各种flex属性,看起来好像是那么回事.

不知道为什么网上为数不多的介绍都没有用到官方demo,大概是官方仓库的编译方式不是gradle吧.

{% asset_img image3.png %}

然而改成gradle编译通过后,发现demo中并没有看到有**动态**的能力.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<YogaLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:yoga="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    yoga:yg_flexDirection="row"
    yoga:yg_alignItems="center"
    >
  <View
      android:layout_width="50dp"
      android:layout_height="50dp"
      android:background="@color/yoga_blue"
      yoga:yg_flex="0"
      yoga:yg_marginAll="5dp"
      />
  <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="@string/child_1_text"
      yoga:yg_flex="0"
      />
  <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="@string/child_2_text"
      yoga:yg_flex="1"
      yoga:yg_marginHorizontal="5dp"
  />
</YogaLayout>
```

真正的实际使用中,不可能直接下发xml到客户端,所以还是需要通过源码看看有没有动态创建YogaLayout的方法.

### 干货!全网独家,使用YogaLayout做真正的动态化.

#### YogaLayout的动态创建和属性配置

Yoga其实是通过每个控件绑定的YogaNode来计算和保存位置信息,YogaLayout的node会在构造方法中自动创建,其动态创建和配置代码如下:
```java
//创建YogaLayout
YogaLayout yogaLayout = new YogaLayout(this);
//拿到内部Node进行容器配置
YogaNode yogaNode = yogaLayout.getYogaNode();
yogaNode.setFlexDirection(YogaFlexDirection.ROW);
//将容器添加到现有的父组件
rootLayout.addView(yogaLayout,new FrameLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT));
```

#### Child的动态创建和属性配置

Child也分YogaLayout和普通控件如TextView之类.但是观察源码得知:在加入YogaLayout的时候,会自动为其创建YogaNode.
```java
    if (child instanceof YogaLayout) {
      childNode = ((YogaLayout) child).getYogaNode();
    } else {
      if(mYogaNodes.containsKey(child)) {
        childNode = mYogaNodes.get(child);
      } else {
        childNode = YogaNode.create();
      }

      childNode.setData(child);
      childNode.setMeasureFunction(new ViewMeasureFunction());
    }
```
因此child的动态配置代码如下:
```java
        //创建基本控件
        textView1 = new TextView(this);
        textView1.setText("动态文案");
        textView1.setGravity(Gravity.CENTER);
        textView1.setTextColor(Color.WHITE);
        textView1.setBackgroundColor(Color.RED);
        //将控件添加到YogaLayout中
        yogaLayout.addView(textView1, -1, new YogaLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT));
        //得到控件对应的YogaNode
        textNode1 = yogaLayout.getYogaNodeForView(textView1);
        //设置Flex属性
        textNode1.setFlex(1);
        textNode1.setAlignSelf(YogaAlign.CENTER);
```
这里和传统的ViewGroup+LayoutParams稍有不同的是,**YogaNode必须在被添加进父容器后再从父容器获取**.
当然YogaLayout也提供了addView(View child, YogaNode node)这样的方法,可以用类似传统方式把child和node一起加进去,但是需要对YogaNode做一些本在addView(View child, int index, ViewGroup.LayoutParams params)中自动完成的操作,比较麻烦,这里就不赘述了.

#### 如何在渲染完成之后,动态修改控件的布局

看完前面的,已经可以完成静态渲染了,然后在原生开发中,不可避免的要将其用于列表中,总所周知,列表的item是复用的,那么YogaLayout能否二次修改其内部布局呢?

这部分研究花了较长时间,甚至在尝试对交叉轴方向容器大小进行动态修改时一度认为YogaLayout存在问题.

直接上结论吧:
##### 对于YogaLayout内部元素的尺寸进行二次修改.使用YogaLayout.invalidate(View view)方法.
```java
        findViewById(R.id.button1).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView1.setText("文案动态修改,加长");
                yogaLayout.invalidate(textView1);
            }
        });
```
哪个元素的内容发生了变化,就调用其parent的invalidate方法.由于其内部实际上是将其node标记为dirty,所以并不用担心性能问题.

##### YogaLayout内部元素变更后,对YogaLayout自身的尺寸进行自动适应.需要将YogaLayout的宽高设置为Float.NaN.
举个例子:
YogaLayout内部的TextView文字大小发生了变化.导致YogaLayout的高度也要跟着变大.此时使用YogaLayout.invalidate(View view)方法仅能将textview的尺寸变大,yogaLayout自身并不会撑高.
{% asset_img image4.png %}
尝试了包括invalidate,requestLayout,calculateLayout等无数方法,都无法改变最外层YogaLayout的高度.
最后发现:将YogaLayout高度设置为Float.NaN即可让其重新计算...
```java
        findViewById(R.id.button1).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView1.setTextSize(30);
                yogaLayout.invalidate(textView1);
                yogaLayout.getYogaNode().setHeight(Float.NaN);
                yogaLayout.getYogaNode().setWidth(Float.NaN);
            }
        });
```

—–
自此,已经可以完成基于YogaLayout的动态渲染.然而这只是UI部分,完整的渲染还需要进行数据绑定.这一部分涉及UI描述文件结构的定义,通过data相对路径+view绑定binder来实现.因篇幅过长,就不介绍了.
