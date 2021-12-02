---
title: Android-Studio：使用Gradle构建不同版本的APP(构建变体)
date: 2016.05.25 14:57
categories: 
    - [Android]
tags: [Android,Android Studio] 
cover: /images/android_banner.webp
---

Android Studio（我使用的Studio版本是2.0）中有一个构建变体的功能，默认位于左下角
{% asset_img 1.webp %}
那这个功能是做什么用的呢，一般来说我们在做项目的时候，可能有这样的需求，一个项目中需要有不同版本，比如说免费版，收费版，周年庆版啦等等，这些版本大部分功能和模块是一样的只是部分不同，以前可能是通过svn或者git上建立分支来进行版本控制，但维护起来很麻烦。所以构建变体这个功能用官方的说法就是你**可以在一个项目里面构建不同的版本**，对，而且打包的时候可以一次性打包所有版本，是不是超级爽。英语好的同学可以看官方的教程[Configure Build Variants](https://developer.android.com/studio/build/build-variants.html)。

<!-- more -->

## 配置Product Flavors 
第一步，在moudle中的build文件中配置Product Flavors

``` java
android {
    defaultConfig { ... }
 //... 
    productFlavors {
        free {
            applicationId "cc.abto.free"
            versionCode 125
            versionName "2.3.5" + ""

        }
        charge {
            applicationId "cc.abto.charge"
            versionCode 7
            versionName "1.0.7" + ""

        }
    }
//...
}
```

上面的代码中，我们创建了两个版本，一个free版和一个charge版，其中defaultConfig 为默认值，productFlavors {}会复写所有可以复写的值。配置好之后我们就可以**Sync Project**一下啦，然后就可以在左下角的Build Variants工具栏上选择版本啦。
{% asset_img 2.webp %}
第二步，我们还需要给每个版本创建对应的文件夹，点击src文件夹右键

{% asset_img 3.webp %}
文件夹的名称和productFlavors {}中的名称对应。然后再在各个文件下常见相应的java和res以及AndroidManifest等文件夹或文件。当然手动创建这些文件夹很麻烦（我就是这么懒），我们可以通过新建一个Activity来创建相关的文件夹。然后在Acitvity的创建界面，我们选择放置在哪一个版本。

{% asset_img 4.webp %}
如上图所示，当当当，需要的文件夹就都创建完成了，Build Variants选择版本和创建相应文件夹之后，在android目录结构下就只能看到这个版本的文件

{% asset_img 5.webp %}
然后你就可以针对这个版本做开发啦，当你要开发另一个版本时，再在Build Variants选择相应的版本，怎么样，是不是很赞，给了这么图是因为这东西跟理论不同，希望能尽量的直观点。


### 不同的Manifest需求
每个版本都可以有自己的清单文件，Manifest可以通过Merge的方式合并多个Manifest源。也就是说，manifest的merge会将每个元素及其子元素的节点和属性进行合并。但是每个版本的manifest之间是不会合并的。

### 不同的依赖
你也可以给不同的版本使用不同的依赖，在build.gradle中，使用Flavor名+Compile来规定特定Flavor所需依赖。

```java
dependencies {
    //...
    freeCompile 'com.android.support:appcompat-v7:23.3.0'
    chargeCompile  'com.android.support:design:23.3.0'
}
```

### 不同ProGuard需求
当然也可以针对版本做不同的混淆要求

```java
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }

    productFlavors {
        flavor1 {
        }
        flavor2 {
            proguardFile 'some-other-rules.txt'
        }
    }
}
```

### 打包
最后，我们在打包发布的时候可以在列表中选择所有版本，gradle就会一次性打包好所有版本。
{% asset_img 6.webp %}
