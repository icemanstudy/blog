---
title: Flutter动画-Animation
date: 2020-08-08 10:23:16
categories:
- 跨平台
index_img:
- /img/post/16.jpg
excerpt:
- Flutter动画执行,其实挺low的.有点像android的属性动画.
---

### 核心类

##### Animation  
动画类,用于获取动画当前的值和状态.

##### Tween 
定义动画值的范围.其默认单位是double,同时也有Tween的不同子类,用于定义不同的动画值类型(例如颜色,对齐方式).  
 
```
animation1 = new Tween<double>(begin: 100, end: 300);
animation2 = new ColorTween(begin: Colors.transparent, end: Colors.black54);
```


##### Curve 
定义Tween的变化速度.类似Interpolator.
```
@override
  double transform(double t) {
    if (t == 0.0 || t == 1.0) {
      return t;
    }
    return super.transform(t);
  }
```


##### AnimationController  
继承自Animation的特殊动画,有两个特征:
- 内部包含一个从0~1的动画取值范围.
- 具备控制动画的方法,例如forward,reverse方法和指定动画时长.  
 
```
controller = new AnimationController(duration: Duration(milliseconds: 2000), vsync: this);
```
vsync参数:通常是混入SingleTickerProviderStateMixin类的某个widget,当该widget不显示时,动画暂停,widget显示时,动画恢复执行,用于避免动画相关UI不在屏幕时消耗资源.


##### CurvedAnimation
另一个继承自Animation的特殊动画,比较方便的指定Tween的变化速度.  

```
curvedAnimation = new CurvedAnimation(parent: controller,curve: Curves.easeInOutBack);
```

### Flutter中的动画
动画本质上是控件的某个样式属性,按照一定的规则进行变化. 
Flutter中将该属性的取值,变化范围和变化速度分开定义,最终组合成动画.

### 动画实现方式
#### 基本实现方式
步骤:
1. 构建Tween类指定动画取值范围(若类似透明度这种,从0~1,可跳过该步骤)
1. 使用CurvedAnimation指定变化曲线(若为线性变化,可跳过该步骤)
1. 使用AnimationController启动动画
1. 实时获取当前动画值
1. 在控件属性上应用

获取动画状态和进度的方式  
```
animation
      ..addStatusListener((status) {
        print("animation当前状态:" + animation.value.toString());
      })
      ..addListener(() {
        print("animation当前值:" + animation.value.toString());
      });
```

在创建Widget时,使用动画当前值作为属性值.

```
      body: Center(
        child: Image(
          image: AssetImage("images/pig.png"),
          width: animation.value,
          height: animation.value,
          fit: BoxFit.fill,
        ),
      )
```
启动动画,在动画值发生变化时,调用StatefulWidget对应State的setState方法,触发Widget的重新build.使其刷新样式.

```
    controller.forward();
    animation
      ..addListener(() {
        setState(() {
          print("animation当前值:" + animation.value.toString());
        });
      });
```
注意:  
在控件被移除时,将动画释放,防止内存泄漏.

```
  @override
  void dispose() {
    super.dispose();
    controller.dispose();
  }
```

#### 使用AnimatedWidget控件实现
系统提供继承自StatefullWidget的AnimatedWidget控件来自动完成addStateListener并在值变化时调用setState的方法.
```
class AnimatedImage extends AnimatedWidget {
  AnimatedImage({Key key, Animation<double> animation})
      : super(key: key, listenable: animation);

  Widget build(BuildContext context) {
    final Animation<double> animation = listenable;
    return new Center(
      child: Image(
          image: AssetImage("images/pig.png"),
          width: animation.value,
          height: animation.value,
          fit: BoxFit.fill,
        )
    );
  }
}
```
构建时传入动画变量,并在内部build方法中使用动画值来创建控件.这样动画在执行时会自动调用setState进而触发build方法重新构建控件.

#### 职责分离
在上述例子中,动画直接植入到控件的build方法中,其child的生成关联在一起.然而实际使用中控件自身的动画应该与其child的展示**解耦**.该控件应该可以在**接收任何child**的同时,正常展示指定动画. 

职责分离:
- 显示图片
- 指定Animation对象
- 渲染过渡效果

AnimatedBuilder继承自AnimatedWidget,在其内部执行addStateListener和setState操作的同时,使用build参数将child做透传,使动画应用于child的构建完全隔离.

```
class ImageTransition extends StatelessWidget {
  ImageTransition({this.child, this.animation});

  final Widget child;
  final Animation<double> animation;

  Widget build(BuildContext context) {
    return new Center(
      child: new AnimatedBuilder(
          animation: animation,
          builder: (BuildContext context, Widget child) {
            return new Container(
                height: animation.value, width: animation.value, child: child);
          },
          child: child),
    );
  }
}
```

#### 系统预置的动画控件
针对常用动画,系统提供了一系列继承自AnimatedWidget的控件来方便开发者实现.例如ScaleTransition,FadeTransition.

例如缩放动画,只用ScaleTransition包装,并指定动画值来源即可.
```
      body: Center(
          child: ScaleTransition(
              scale: animation,
              child: Image(
                image: AssetImage("images/pig.png"),
              ))),
```
透明度动画同理:  

```
      body: Center(
          child: FadeTransition(
              opacity: animation,
              child: Image(
                image: AssetImage("images/pig.png"),
              ))),
```


