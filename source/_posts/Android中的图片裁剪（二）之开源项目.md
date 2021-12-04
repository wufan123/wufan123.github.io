---
title: Android中的图片裁剪（二）之开源项目
date: 2016.05.19 16:32
categories: 
    - [Android]
tags: [Android]
---

在上一篇博客[Android中的图片裁剪（一）](/2016/05/18/Android中的图片裁剪（一）之系统裁剪工具)中，简单介绍了一下使用系统自带的裁剪软件实现图片裁剪功能。可是有时候系统自带的裁剪软件不能满足项目需求的时候,只能用三方的或者自己写一个了。
<!-- more -->

## 第三方开源项目
都说不要重复造轮子么，github上有很多优秀的开源项目：
- [android-crop](https://github.com/jdamcd/android-crop)
在github搜crop这个项目是star最多的，我之前做的一个应用就是集成的这个开源项目，感觉还是不错的哈，直接拿来用还是挺方便的，在上面做二次开发也是挺快的。下面是效果图：
{% asset_img 1.webp %}

- [Cropper](https://github.com/edmodo/cropper)
这个不仅能裁剪还能旋转哦，这个我也用过感觉也挺好的，集成也挺快的，能快速定制包括裁剪框在内的几个基础属性。下面是效果图：
{% asset_img 2.webp %}

- [itmap Smart Clipping using OpenCV](https://github.com/beartung/tclip-android)
能自动识别图片中的重要区域，并且在图片裁剪时保留重要区域，比如说人脸识别啥的，还能自动识别其它重要区域。如果图片中未识别出人脸，则会根据特征分布计算出重区域 。下面是效果图：

{% asset_img 3.webp %}

- 其他还有很多像[cropimage](https://github.com/biokys/cropimage)，[SimpleCropView](https://github.com/IsseiAoki/SimpleCropView)，大家可以上github上自行查找。

<br>

## 最后
大部分情况下集成这些优秀的开源项目就可完成需求了。但在需要高度定制裁剪功能的时候，要怎么实现呢，下一篇文章中，就和大家探讨一下裁剪图片实现的原理。
