---
title: android上实现卡片阴影的正确姿势
date: 2020-01-03 18:38:04
categories:
- android
index_img:
- /img/post/5.jpg
excerpt:
- android 4.4适配,你懂的.
---

列表中的卡片式布局如今已经司空见惯了.通常来说卡片式布局有以下特征:

* 有圆角
* 边距加大
* 边框有阴影

圆角可以用**GradientDrawable**的Corner相关api实现.
边距什么的太简单就不说了.
最麻烦的要数阴影.先说说市面上常用的方式存在的问题:

* 使用**CardView**包装.
	1.仅支持5.0及以上.
	2.不支持修改阴影颜色,在暗夜模式下无法显示.

* 使用**View.setElevation**方法.同CardView.

* 使用自定义shape+.9图片.
	1.后期修改麻烦.
	2..9图片和shape组合会有莫名其妙的layout问题.
	3..9图片的存在会影响性能.

多方尝试无果之后,决定自己绘制阴影.

**由于阴影实际上绘制在容器以外,以下代码有一个要求:其父容器必须设置android:clipChildren="false"**

#### 首先为了方便使用,增加一个自定义容器,继承自FrameLayout即可.

```java
public class MgCardContainer extends FrameLayout {
    private Context context;
    /**关闭卡片样式*/
    public static final int CARD_FLAG_DISABLE = 0;
    /**卡片头*/
    public static final int CARD_FLAG_TOP = 1;
    /**卡片中间*/
    public static final int CARD_FLAG_CENTER = 2;
    /**卡片底部*/
    public static final int CARD_FLAG_BOTTOM = 3;
    /**卡片四周*/
    public static final int CARD_FLAG_ALL_AROUND = 4;
    private Paint mPaint;
    /**设置单独卡片时使用的rect*/
    private RectF mRectF;
    /**背景颜色,白天和晚上是不同的*/
    private int backGroundColor;
    /**阴影颜色,白天和晚上使用同一个颜色*/
    private int shadowColor;
    /**卡片样式,有0,1,2,3,4几种.*/
    private int cardFlag = 0;
    /**卡片的圆角值*/
    private int cornerRadius = 0;
    /**阴影大小*/
    private int shadowWidth;
    /**用来绘制阴影的画笔粗细*/
    private int paintWidth;

    public MgCardContainer(@NonNull Context context) {
        super(context);
        this.context = context;
        initView();
    }

    public MgCardContainer(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        this.context = context;
        initView();
    }

    public MgCardContainer(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.context = context;
        initView();
    }

    private void initView() {
        paintWidth = ScreenUtil.dip2px(context, 1);
        shadowWidth = ScreenUtil.dip2px(context, 3);
        cornerRadius = ScreenUtil.dip2px(context, 3);
        backGroundColor = getResources().getColor(R.color.color_v60_bg_with_card);
        shadowColor = Color.GRAY;
        mPaint = new Paint();
        mPaint.setColor(backGroundColor);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(paintWidth);
        mPaint.setShadowLayer(shadowWidth, 0, 0, shadowColor);
        mRectF = new RectF();
    }
```
#### 重写draw方法,在调用super之前,插入绘制阴影逻辑.
```java
@Override
    public void draw(Canvas canvas) {
        if (cardFlag != 0) {
            int height = getHeight();
            int width = getWidth();
            switch (cardFlag) {
                case CARD_FLAG_TOP:
                    drawHeadShadow(canvas, width, height);
                    break;
                case CARD_FLAG_CENTER:
                    drawLeftAndRightShadow(canvas, width, height);
                    break;
                case CARD_FLAG_BOTTOM:
                    drawBottomShadow(canvas, width, height);
                    break;
                case CARD_FLAG_ALL_AROUND:
                    drawAllAroundShadow(canvas, width, height);
                    break;
            }
        }
        super.draw(canvas);
    }
```

#### 针对卡片头部,卡片中间部分,卡片底部,单个卡片,分别进行阴影绘制.

头部阴影
```java
    /**
     * 画卡片头部阴影
     */
    private void drawHeadShadow(Canvas canvas, int width, int height) {
        mPaint.setShadowLayer(shadowWidth, 0, 0, shadowColor);
        //顶部阴影
        canvas.drawLine(cornerRadius, paintWidth, width - cornerRadius, paintWidth, mPaint);

        mPaint.setShadowLayer(shadowWidth, 0, -shadowWidth, shadowColor);
        //左侧阴影
        canvas.drawLine(paintWidth, cornerRadius, paintWidth, height + cornerRadius + shadowWidth, mPaint);
        //右侧阴影
        canvas.drawLine(width - paintWidth, cornerRadius, width - paintWidth, height + cornerRadius + shadowWidth, mPaint);
    }
```
中间阴影
```java
    /**
     * 画左右两边阴影
     */
    private void drawLeftAndRightShadow(Canvas canvas, int width, int height) {
        mPaint.setShadowLayer(shadowWidth, 0, shadowWidth, shadowColor);
        //左侧阴影
        canvas.drawLine(paintWidth, 0, paintWidth, height, mPaint);
        //右侧阴影
        canvas.drawLine(width - paintWidth, 0, width - paintWidth, height, mPaint);
    }
```
底部阴影
```java
    /**
     * 画底部卡片的阴影
     */
    private void drawBottomShadow(Canvas canvas, int width, int height) {
        mPaint.setShadowLayer(shadowWidth, 0, 0, shadowColor);
        //底部阴影
        canvas.drawLine(cornerRadius, height - paintWidth, width - cornerRadius, height - paintWidth, mPaint);

        mPaint.setShadowLayer(shadowWidth, 0, shadowWidth, shadowColor);
        //左侧阴影
        canvas.drawLine(paintWidth, 0, paintWidth, height - cornerRadius, mPaint);
        //右侧阴影
        canvas.drawLine(width - paintWidth, 0, width - paintWidth, height - cornerRadius, mPaint);
    }
```
单个卡片四周绘制阴影
```java
    /**
     * 画单个卡片的阴影
     */
    private void drawAllAroundShadow(Canvas canvas, int width, int height) {
        mPaint.setShadowLayer(shadowWidth, 0, 0, shadowColor);
        mRectF.left = paintWidth;
        mRectF.top = paintWidth;
        mRectF.right = width - paintWidth;
        mRectF.bottom = height - paintWidth;
        canvas.drawRoundRect(mRectF, cornerRadius, cornerRadius, mPaint);
    }
```

**之所以分成四个方法,而不是用组合的方式,是因为中间涉及阴影偏移,阴影线条长度有一些细微差异.可看代码自行感受.**

#### 顺便把自动增加margin和圆角的方法发上来.
```java
    /**
     * 开启卡片样式
     * @param cardFlag {@link MgCardContainer#CARD_FLAG_TOP},
     * {@link MgCardContainer#CARD_FLAG_CENTER}{@link MgCardContainer#CARD_FLAG_BOTTOM}{@link MgCardContainer#CARD_FLAG_ALL_AROUND}
     */
    public void enableCardStyle(int cardFlag) {
        this.cardFlag = cardFlag;
        MarginLayoutParams pa = (MarginLayoutParams) getLayoutParams();
        //第一步,先设置margin
        if (getTag(R.id.dsl_tag_view_card) == null) {
            setTag(R.id.dsl_tag_view_card, cardFlag);
            Drawable drawable = getBackground();
            GradientDrawable gradientDrawable;
            if (drawable instanceof GradientDrawable) {
                gradientDrawable = (GradientDrawable) drawable;
            } else {
                gradientDrawable = new GradientDrawable();
                gradientDrawable.setColor(backGroundColor);
            }
            pa.leftMargin += ScreenUtil.dip2px(context, 6);
            pa.rightMargin += ScreenUtil.dip2px(context, 6);
            switch (cardFlag) {
                case 1:
                    pa.topMargin += ScreenUtil.dip2px(context, 6);
                    gradientDrawable.setCornerRadii(new float[]{cornerRadius, cornerRadius, cornerRadius, cornerRadius, 0, 0, 0, 0});
                    break;
                case 2:
                    gradientDrawable.setCornerRadius(0);
                    break;
                case 3:
                    pa.bottomMargin += ScreenUtil.dip2px(context, 6);
                    gradientDrawable.setCornerRadii(new float[]{0, 0, 0, 0, cornerRadius, cornerRadius, cornerRadius, cornerRadius});
                    break;
                case 4:
                    pa.topMargin += ScreenUtil.dip2px(context, 6);
                    pa.bottomMargin += ScreenUtil.dip2px(context, 6);
                    gradientDrawable.setCornerRadius(cornerRadius);
                    break;
            }
            setBackground(gradientDrawable);
            setLayoutParams(pa);
            invalidate();
        }
    }

    /**
     * 禁用卡片样式
     */
    public void disableCardStyle() {
        cardFlag = 0;
        MarginLayoutParams pa = (MarginLayoutParams) getLayoutParams();
        if (getTag(R.id.dsl_tag_view_card) != null) {
            int flag = (int) getTag(R.id.dsl_tag_view_card);
            pa.leftMargin -= ScreenUtil.dip2px(context, 6);
            pa.rightMargin -= ScreenUtil.dip2px(context, 6);
            switch (flag) {
                case 1:
                    pa.topMargin -= ScreenUtil.dip2px(context, 6);
                    break;
                case 3:
                    pa.bottomMargin -= ScreenUtil.dip2px(context, 6);
                    break;
                case 4:
                    pa.topMargin -= ScreenUtil.dip2px(context, 6);
                    pa.bottomMargin -= ScreenUtil.dip2px(context, 6);
                    break;
            }
            setTag(R.id.dsl_tag_view_card, null);
            setBackgroundColor(Color.TRANSPARENT);
            setLayoutParams(pa);
            invalidate();
        }
    }
```

就这样了.展示效果完美,适配系统暗夜模式,无毛边,无越界.