---
title: Android中全局异常的捕获
date: 2016.06.03 14:56
categories: 
    - [Android]
tags: [Android]
cover: /images/android_banner.webp
---

应用的crash是让人很蛋疼的问题，在开发测试的时候还能根据日志输出什么的进行排查修复，但是应用发布以后，用户的随意性访问出现测试时未知的Bug导致我们的程序crash，此时我们是无法直接获取的错误log的，也就无法修复Bug。所以这时候我们就需要一个能全局捕获异常，并且将这个异常信息上传到服务器的功能，以便根据收集到的异常信息，在后期的版本中进行修复，改善用户体验。

<!-- more -->

## UncaughtExceptionHandler
实现这个功能我们只要自定义一个实现了Thread.UncaughtExceptionHandler接口的异常处理类，并在应用初始化的时候注册这个类就可以了。
``` java
public class CrashHandler implements Thread.UncaughtExceptionHandler
{

    private static CrashHandler ourInstance = new CrashHandler();

    public static CrashHandler getInstance()
    {
        return ourInstance;
    }

    private CrashHandler()
    {
    }

    @Override
    public void uncaughtException(Thread thread, Throwable ex)
    {
     //TODO
    }
}
```
这边呢，我们一般将CrashHandler写成单例模式，重写上面的uncaughtException方法自定义对异常的处理，然后呢，Application或者Activity的onCreate方法里注册这个异常处理类就可以了
``` java
Thread.setDefaultUncaughtExceptionHandler(CrashHandler.getInstance());
```
这边呢再贴一点uncaughtException中处理异常的代码，给大家参考一下
``` java
public void uncaughtException(Thread thread, Throwable ex)
{

    
    dumpEx2SdCard(ex);//将错误日志导入到SD卡中
    upEx2server();//将日志上传到服务器
    if (mDefaultHandler != null)
    {
        mDefaultHandler.uncaughtException(thread, ex);
    }
    else
    {
        Process.killProcess(Process.myPid());//做一些退出或者提醒处理
    }
}

private void dumpEx2SdCard(Throwable ex)
{
         //...
        PrintWriter printWriter = new PrintWriter(new BufferedWriter(new FileWriter(file)));
        PackageInfo packageInfo = context.getPackageManager().getPackageInfo(context.getPackageName(), PackageManager.GET_ACTIVITIES);
        ex.printStackTrace(printWriter);
        printWriter.print(ex.getCause());
        printWriter.print(time);
        printWriter.print(ex.getMessage());
        printWriter.print("APP版本："+ packageInfo.versionName+"_"+packageInfo.versionCode);
        printWriter.print("android 版本："+ Build.VERSION.RELEASE+"_"+Build.VERSION.SDK_INT);
        printWriter.print("制造商："+Build.MANUFACTURER);
        //...
}
```
一般这边的代码都是将错误日志和设备信息写到SD卡中然后再上传到服务器中，具体实现就不贴那么多的代码啦，注重流程就是啦。
##开发者服务
集成一些第三方服务可以大大的加快我们的开发速度，诸如此类的统计功能我们只要集成一些像百度云推送，极光推送，友盟+等等的第三方SDK，然后做一些初始化，几行代码就搞定了。而且这些平台提供的各种强大方便的服务，确实是我们开发者的福音。像下图展示的友盟应用管理中心，各种统计功能基本上都能满足我们的需求了。

{% asset_img 1.webp %}

再推荐一个网站。[DevStore [移动互联网企业运营解决方案整合平台](http://www.baidu.com/link?url=upIpM9HcrAOsbxadr2T4f4BMfV6mi2FG54BHlfkzdTmq3BtE6gLYr5PB0rCTTQJ_&wd=&eqid=b3a05c1a00012bae00000005575122a2)](http://www.devstore.cn/)这里面总结了各种强大的第三方服务平台，建议收藏哦。
