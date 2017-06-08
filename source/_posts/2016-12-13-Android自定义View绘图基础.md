---
title: Android自定义View绘图基础
date: 2016-12-13 16:16:52
categories:
tags:
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.com/2016/12/13/2016-12-13-Android%E8%87%AA%E5%AE%9A%E4%B9%89View%E7%BB%98%E5%9B%BE%E5%9F%BA%E7%A1%80/) | [CSDN](http://blog.csdn.net/q4878802/article/details/53610590)

---


# Android自定义View绘图基础

## View的测量

 控件的测量可以说是固定写法，原生的View只支持`EXACTLY`的测量模式，我们自定义的控件可以重写`onMeasure`方法

``` java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getMeasuredSize(widthMeasureSpec), getMeasuredSize(heightMeasureSpec));
}
```

onMeasure方法给我们返回的`widthMeasureSpec`和`heightMeasureSpec`，我们并不能直接拿过来使用，需要使用`MeasureSpec`类进行解析，来获取测量后的具体值。
首先需要获取测量模式

``` java
MeasureSpec.getMode(measureSpec)
``` 

getMode方法返回是测量的模式，有以下3种类型：
- MeasureSpec.EXACTLY : 精确值模式（指定值/match_parent）
- MeasureSpec.AT_MOST : 最大值模式（wrap_content）
- MeasureSpec.UNSPECIFIED : 不指定大小的测量模式（一般用不上）

获取到了测量模式以后，获取测量后的大小

``` java
MeasureSpec.getSize(measureSpec)
```

根据上面的意思，可以封装我们的`getMeasuredSize`方法

``` java
// 默认大小
private static final int DEFAULT_SIZE = 200;

/**
 * 获取测量后的大小
 *
 * @param measureSpec measureSpec
 * @return measuredSize
 */
private int getMeasuredSize(int measureSpec) {
    switch (MeasureSpec.getMode(measureSpec)) {
        case MeasureSpec.EXACTLY: // 精确值模式（指定值/match_parent）
            Log.i(TAG, "getMeasuredSize: 精确值模式");
            return MeasureSpec.getSize(measureSpec);
        case MeasureSpec.AT_MOST: // 最大值模式（wrap_content）
            Log.i(TAG, "getMeasuredSize: 最大值模式");
            return Math.min(DEFAULT_SIZE, MeasureSpec.getSize(measureSpec));
        case MeasureSpec.UNSPECIFIED: // 不指定大小的测量模式
            return DEFAULT_SIZE;
        default:
            return DEFAULT_SIZE;
    }
}
```

现在我们自定义的View就支持自定义的大小了，包括`match_parent`、`wrap_content`、`具体值`。


## View的绘制

### 画笔属性

创建画笔
``` java
Paint paint = new Paint();
```

|  方法 | 描述 | 举例 |
| ------ | ------| -------|
| public void setAntiAlias(boolean aa) | 设置画笔锯齿效果| true 无锯齿效果   |
| public void setColor(@ColorInt int color) | 设置画笔颜色 |    |
| public void setARGB(int a, int r, int g, int b) | 设置画笔的A、R、G、B值 |    |
| public void setAlpha(int a) | 设置画笔的Alpha值 |    |
| public void setTextSize(float textSize) | 设置字体的尺寸 |    |
| public void setStyle(Style style) | 设置画笔的风格（空心或实心） | paint.setStyle(Paint.Style.STROKE);// 空心    paint.setStyle(Paint.Style.FILL); // 实心 |
| public void setStrokeWidth(float width) | 设置空心边框的宽度 |    |

### Shader

BitmapShader, ComposeShader, LinearGradient, RadialGradient, SweepGradient

### 点

``` java
/**
 * Helper for drawPoints() for drawing a single point.
 */
public void drawPoint(float x, float y, @NonNull Paint paint)
```
``` java
canvas.drawPoint(10, 10, paint);
```

### 直线

绘制一条直线
``` java
/**
 * Draw a line segment with the specified start and stop x,y coordinates,
 * using the specified paint.
 *
 * <p>Note that since a line is always "framed", the Style is ignored in the paint.</p>
 *
 * <p>Degenerate lines (length is 0) will not be drawn.</p>
 *
 * @param startX The x-coordinate of the start point of the line
 * @param startY The y-coordinate of the start point of the line
 * @param paint  The paint used to draw the line
 */
public void drawLine(float startX, float startY, float stopX, float stopY, @NonNull Paint paint)
```

绘制多条直线
``` java
public void drawLines(@Size(multiple=4) @NonNull float[] pts, @NonNull Paint paint)
```

### 矩形

``` java
/**
 * Draw the specified Rect using the specified paint. The rectangle will
 * be filled or framed based on the Style in the paint.
 *
 * @param left   The left side of the rectangle to be drawn
 * @param top    The top side of the rectangle to be drawn
 * @param right  The right side of the rectangle to be drawn
 * @param bottom The bottom side of the rectangle to be drawn
 * @param paint  The paint used to draw the rect
 */
public void drawRect(float left, float top, float right, float bottom, @NonNull Paint paint)
```

``` java
canvas.drawRect(100, 100, 200, 200, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213155250067?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 圆角矩形
``` java
/**
 * Draw the specified round-rect using the specified paint. The roundrect
 * will be filled or framed based on the Style in the paint.
 *
 * @param rx    The x-radius of the oval used to round the corners
 * @param ry    The y-radius of the oval used to round the corners
 * @param paint The paint used to draw the roundRect
 */
public void drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, @NonNull Paint paint)
```

``` java
canvas.drawRoundRect(100, 100, 200, 200, 20, 20, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213155334255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 圆
``` java
/**
 * Draw the specified circle using the specified paint. If radius is <= 0,
 * then nothing will be drawn. The circle will be filled or framed based
 * on the Style in the paint.
 *
 * @param cx     The x-coordinate of the center of the cirle to be drawn
 * @param cy     The y-coordinate of the center of the cirle to be drawn
 * @param radius The radius of the cirle to be drawn
 * @param paint  The paint used to draw the circle
 */
public void drawCircle(float cx, float cy, float radius, @NonNull Paint paint)
```
 
``` java
// 画圆
canvas.drawCircle(200, 200, 100, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213160033632?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 扇形

``` java
/**
 * <p>Draw the specified arc, which will be scaled to fit inside the
 * specified oval.</p>
 *
 * <p>If the start angle is negative or >= 360, the start angle is treated
 * as start angle modulo 360.</p>
 *
 * <p>If the sweep angle is >= 360, then the oval is drawn
 * completely. Note that this differs slightly from SkPath::arcTo, which
 * treats the sweep angle modulo 360. If the sweep angle is negative,
 * the sweep angle is treated as sweep angle modulo 360</p>
 *
 * <p>The arc is drawn clockwise. An angle of 0 degrees correspond to the
 * geometric angle of 0 degrees (3 o'clock on a watch.)</p>
 *
 * @param startAngle Starting angle (in degrees) where the arc begins
 * @param sweepAngle Sweep angle (in degrees) measured clockwise
 * @param useCenter If true, include the center of the oval in the arc, and
                    close it if it is being stroked. This will draw a wedge
 * @param paint      The paint used to draw the arc
 */
public void drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint)
```

``` java
// 扇形 起始角度0度 旋转角度120度
canvas.drawArc(100, 100, 200, 200, 0, 120, true, paint);
```
![这里写图片描述](http://img.blog.csdn.net/20161213160316574?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 弧形

扇形通过调整属性，可以实现弧形的效果，首先画笔设置成空心

``` java
// 空心
paint.setStyle(Paint.Style.STROKE);
```

设置画扇形的`useCenter`参数为`false`

``` java
// 弧形 起始角度0度 旋转角度120度
canvas.drawArc(100, 100, 200, 200, 0, 120, false, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213160433902?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 椭圆

``` java
/**
 * Draw the specified oval using the specified paint. The oval will be
 * filled or framed based on the Style in the paint.
 */
public void drawOval(float left, float top, float right, float bottom, @NonNull Paint paint)
```

``` java
// 椭圆
canvas.drawOval(100, 100, 500, 300, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213160532467?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 文字

``` java
/**
 * Draw the text, with origin at (x,y), using the specified paint. The
 * origin is interpreted based on the Align setting in the paint.
 *
 * @param text  The text to be drawn
 * @param x     The x-coordinate of the origin of the text being drawn
 * @param y     The y-coordinate of the baseline of the text being drawn
 * @param paint The paint used for the text (e.g. color, size, style)
 */
public void drawText(@NonNull String text, float x, float y, @NonNull Paint paint)
```

``` java
// 文字
paint.setTextSize(30);
paint.setColor(Color.BLACK);
canvas.drawText("KongQingwei", 100, 100, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213160730434?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在指定位置显示每个字符

``` java
/**
 * Draw the text in the array, with each character's origin specified by
 * the pos array.
 *
 * @param text  The text to be drawn
 * @param pos   Array of [x,y] positions, used to position each character
 * @param paint The paint used for the text (e.g. color, size, style)
 *
 * @deprecated This method does not support glyph composition and decomposition and
 * should therefore not be used to render complex scripts. It also doesn't
 * handle supplementary characters (eg emoji).
 */
@Deprecated
public void drawPosText(@NonNull String text, @NonNull @Size(multiple=2) float[] pos, @NonNull Paint paint)
```

``` java
canvas.drawPosText("KongQingwei", new float[]{
        30,30,
        60,60,
        90,90,
        120,120,
        150,150,
        180,180,
        210,210,
        240,240,
        270,270,
        300,300,
        330,330
}, paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213160854809?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 绘制路径

可以自定义画出想要的任意图形

五角星

``` java
/**
 * Draw the specified path using the specified paint. The path will be
 * filled or framed based on the Style in the paint.
 *
 * @param path  The path to be drawn
 * @param paint The paint used to draw the path
 */
public void drawPath(@NonNull Path path, @NonNull Paint paint)
```

``` java
Path path = new Path();
path.moveTo(0,100);
path.lineTo(250,300);
path.lineTo(150,0);
path.lineTo(50,300);
path.lineTo(300,100);
path.lineTo(0,100);
canvas.drawPath(path,paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213161030317?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

空心

``` java
paint.setStyle(Paint.Style.STROKE);
```

![这里写图片描述](http://img.blog.csdn.net/20161213161158686?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 图形裁剪

以上面的实心五角星为例，以五角星的中心为圆心，裁剪出一个半径为100的圆形

``` java
/**
    * Modify the current clip with the specified path.
 *
 * @param path The path to operate on the current clip
 * @param op   How the clip is modified
 * @return     true if the resulting is non-empty
 */
public boolean clipPath(@NonNull Path path, @NonNull Region.Op op)
```

``` java
// 裁剪一个圆形
Path path = new Path();
path.reset();
path.addCircle(150, 150, 100, Path.Direction.CCW);
canvas.clipPath(path, Region.Op.INTERSECT);
// canvas.save();
// 画五角星
path.reset();
path.moveTo(0,100);
path.lineTo(250,300);
path.lineTo(150,0);
path.lineTo(50,300);
path.lineTo(300,100);
path.lineTo(0,100);
canvas.drawPath(path,paint);
```

![这里写图片描述](http://img.blog.csdn.net/20161213161327462?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

| Region.Op | 描述 | 样式 |
| --------- | ---- | ------|
| INTERSECT | 裁剪内部交集 | ![这里写图片描述](http://img.blog.csdn.net/20161213161327462?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) |
| DIFFERENCE | 外部      | ![这里写图片描述](http://img.blog.csdn.net/20161213161401588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ4Nzg4MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) |
