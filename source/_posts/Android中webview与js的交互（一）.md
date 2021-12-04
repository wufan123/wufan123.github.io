---
title: Android中webview与js的交互（一）
date: 2016.05.17 16:51
categories: 
    - [Android]
tags: [Android]
---

## webview中调用js
``` java
webView.getSettings().setJavaScriptEnabled(true);//设置js脚本可用
webView.loadUrl("url");//加载页面
webView.loadUrl("javascript:test(a,b,c)");//调用js方法
```
<!-- more -->
## html中调用java
### android中配置
``` java
webView.addJavascriptInterface(new MyJSInterface(),"app");//添加js脚本接口

class  MyJSInterface
{   
 
@JavascriptInterface
public void androidMethod() //提供给js调用的方法
  {
  //todo    
  }
}
```
### html中使用
``` HTML
<div id='b'>
<a onclick="window.app.androidMethod()">b.c</a>
</div>
```
## 总结
  webview与js的交互的基本使用方法如上，补充几点：
- 在这篇[博客](http://jiajixin.cn/2014/09/16/webview-js-safety/)中抄出这段话 
Android Webview有两个非常知名的漏洞:
1、最近爆出来的UXSS漏洞，可以越过同源策略，获得任意网页的Cookie等信息，Android 4.4以下都有此问题，基本无解，只能重新编译浏览器内核解决，详情可以参考[最近移动安全三两事](http://zhuanlan.zhihu.com/fooying/19840752)，感兴趣的可以去看一下[@RAyH4c](http://weibo.com/rayh4c)劫持微博、QQ空间的视频。
2、成名已久的任意命令执行漏洞，通过addJavascriptInterface方法，Js可以调用Java对象方法，通过反射机制，Js可以直接获取Runtime，从而执行任意命令。Android 4.2以上，可以通过声明**@JavascriptInterface**保证安全性，4.2以下不能再调用addJavascriptInterface，需要另谋他法。

- 在实际的混合应用开发中需要对js交互做一些封装设计以便于后期的维护扩展，我会在[Android中webview与js的交互（二）](http://www.jianshu.com/p/cafd518e3aae)中分享一下我的经验
