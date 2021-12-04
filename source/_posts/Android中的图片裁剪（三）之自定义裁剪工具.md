---
title: Android中的图片裁剪（三）之自定义裁剪工具
date: 2016.05.24 16:06
categories: 
    - [Android]
tags: [Android]
---

在上一篇文章中[Android中的图片裁剪（二）之开源项目](/2016/05/19/Android中的图片裁剪（二）之开源项目)介绍了一些优秀的图片裁剪开源项目，在我们实现自己的裁剪功能的时候，也可以看下这些开源项目的源码，看看大牛们都是怎么实现的。

<!-- more -->

## createBitmap方法
首先要做图片裁剪的功能，要先认识Bitmap，在这个类里面，有几个方法可以帮助我们实现功能。就比如下面这一个方法
``` java
/**
 * Returns an immutable bitmap from the specified subset of the source
 * bitmap. The new bitmap may be the same object as source, or a copy may
 * have been made. It is initialized with the same density as the original
 * bitmap.
 *...
 */
public static Bitmap createBitmap(Bitmap source, int x, int y, int width, int height)
```
好吧，方法注解太长，被我省略掉一些了，但是用起来其实挺简单的，只要传入指定的范围，它就能生成一个新的位图给你。（这里先说一句，图片的裁剪功能其实做起来不难，要做好就挺麻烦的，在实际的工具开发中免不了要自定义View这样的工作），而这边的范围一般是一个可拖动的方框，点击裁剪按钮的时候生成我们需要的图片。当然createBitmap方法还有很多其他重载实现，这里就不一一贴出来了。在具体实现的时候我们可以自定义两个View
```java
public class CropImageView extends ImageView {

    HighlightView hv;
    //...
 
}

class HighlightView {
    //...
}
```
这里只是简单的提供一下思路，HighlightView用来表示裁剪框对象，封装裁剪框的一些信息啊，比如宽高和触摸状态什么的。CropImageView继承ImageView用来显示图片和处理触摸事件，在触摸事件中改变HighlightView的状态：
```java
public boolean onTouchEvent(@NonNull MotionEvent event) {
    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
            //对触摸点进行一些逻辑判断，比如摸到裁剪框的边还是其他什么的
            int edge = hv.getHit(event.getX(), event.getY());
            //...
            //然后根据逻辑判断结果设置裁剪框的状态
            hv.setMode((edge == HighlightView.MOVE)
                        ? HighlightView.ModifyMode.Move
                        : HighlightView.ModifyMode.Grow);
            //...
        break;
            //...
```
啦啦啦，是不是说了跟没说一样，主要是具体实现起来真的挺麻烦的，贴那么多代码出来也没啥用，我自己都要看晕了，总归自己造轮子还是挺辛苦的，而且刚造出来也肯定有一些问题的，所以才有那句名言么，不要重复造轮子，我觉得只要看懂源代码的思路，然后根据需求，在别人的轮子上进行改造，这样就快的多啦。


## Matrix的基本使用
一般呢，在裁剪的工具中，在对图片裁剪前，多可以对图片进行一些缩放，平移，旋转之类的操作，这些效果都是可以通过Matrix来实现的，那这个类怎么用呢。我们先看一下[官方文档中Matrix](file:///D:/Android/sdk/docs/reference/android/graphics/Matrix.html)的介绍（自带梯子） ，首先看一下它的官方定义：

**The Matrix class holds a 3x3 matrix for transforming coordinates. **

直译过来呢，就是**一个转换坐标的3x3矩阵**，你妹啊，什么是3x3矩阵...我也不知道啊 ，高等数学这东西我早就扔了好么，不过不知道没关系，面向对象么，我先知道怎么用就可以了，有时间有闲情再去深究。我们先主要看一下Matrix类里面，我们要用到的一些方法(补充说一下，这Matrix类是在graphics包下的，不要导成opengl包下的)：
``` java
/**
 * Postconcats the matrix with the specified translation.
 * M' = T(dx, dy) * M
 */
public boolean postTranslate(float dx, float dy)

/**
 * Postconcats the matrix with the specified rotation.
 * M' = R(degrees) * M
 */
public boolean postRotate(float degrees)
```
好啦，毕竟是文档找的到东西，我这里就只贴两个方法出来，看注解应该很容易理解方法的使用，**第一个方法就是对矩阵进行左乘T(dx, dy)，第二个方法就是对矩阵左乘R(degrees)**，通过源码我们可以看到，方法里面通过jni调用lib层用c，c++实现运算转换的。反正直白点讲，在对它进行图片变换的时候，**第一个方法就是平移图片，第二个方法就是旋转图片**，啦啦啦，其他方法大家可以自行查找文档，问题都不大。

## 图片的平移，缩放，旋转
通过重写View或者Activity的onTouch事件来进行图片的平移，缩放和旋转会比较方便，这里我们要先做一些初始化工作，比如获取图片之类的，一般我们将图片的大小控制在手机内存的1/8，防止OOM和卡顿，然后就是在onTouch事件里做具体实现：
``` java
public boolean onTouch(View v, MotionEvent event)
{

    switch (event.getAction() & MotionEvent.ACTION_MASK)
    {
        // 主点按下
        case MotionEvent.ACTION_DOWN:
            cachMatrix.set(matrix);//用来变化的矩阵
            prev.set(event.getX(), event.getY());
            mode = DRAG;
            break;
        //...
        case MotionEvent.ACTION_MOVE:
            if (mode == DRAG)//拖动
            {
                matrix.set(cachMatrix);
                matrix.postTranslate(event.getX() - prev.x, event.getY() - prev.y);
            }
            else if (mode == ZOOM)//缩放
            {
                float newDist = spacing(event);
                if (newDist > 10f)
                {
                    matrix.set(cachMatrix);
                    float tScale = newDist / dist;
                    matrix.postScale(tScale, tScale, mid.x, mid.y);
                }
            }
            break;
    }
    imgView.setImageMatrix(matrix);
    //...
    return true;
}
```
代码就放了部分上去，主要说明一下思路，当然在将这些功能和裁剪进行合并的时候，是要对裁剪框的位置再进行计算的,比如因为对图片缩放了，所以再返回剪裁框大小的时候是要乘上缩放值的。
``` java
public Rect getScaledCropRect(float scale) {
    return new Rect((int) (cropRect.left * scale), (int) (cropRect.top * scale),
            (int) (cropRect.right * scale), (int) (cropRect.bottom * scale));
}
```
## 总结
图片的裁剪就先写到这里，主要介绍了一些关键方法和整体的实现思路，后面有时间的话，我会把代码整理后发布到github上去的，到时候再来补链接。
