---
title: android-studio-实用技巧
date: 2016.10.10 17:31
categories: 
    - [Android]
tags: [Android,Android Studio]
cover: /images/android_banner.webp
---

- **打jar包**
android studio 默认提供的是aar包，如果要打成jar包的话，就要自己写gradle task，在库工程的build.gradle文件中添加一下代码：
``` java
task makeJar(type: Copy) {
   delete 'build/libs/mysdk.jar'
   from('build/intermediates/bundles/release/')
   into('build/libs/')
   include('classes.jar')
   rename ('classes.jar', 'mysdk.jar')
}
makeJar.dependsOn(build);
``` 
然后在终端执行命令打出jar包
``` bash
gradlew makeJar
``` 
- **引用aar包**
将aar拷贝至lib目录，在module中的build.gradle文件中添加以下代码
``` java
repositories { flatDir { dirs 'libs' } }
dependencies {
compile(name: 'aar_name', ext: 'aar')
}
```
