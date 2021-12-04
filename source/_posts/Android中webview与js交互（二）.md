---
title: Android中webview与js交互（二）
date: 2016.05.18 15:23
categories: 
    - [Android]
tags: [Android]
cover: /images/android_banner.webp
---

在[Android中webview与js的交互（一）](/2016/05/17/Android中webview与js的交互（一）)中简单介绍了webview与js交互的基本使用方法，接下来，为了方便后期的维护扩展，在这里分享一下我自己的一些经验。
<!-- more -->
## JS接口对象
依据依赖倒转原则，这里我们定义一个js接口类，来统一管理webview与js之间交互的接口，例子如下：
``` java
public interface JavaScript{

@JavascriptInterface    
public String getManufacturer(); 

@JavascriptInterface    
public String getVersionName();    
//...
}
```
然后再创建一个抽象基类，实现一些基础共用方法：
``` java
public abstract  class BaseJavaScript 
{

    private WebView webView;

    @Override
    public  final JavaScript setWebView(WebView webView, String name)
    {
        this.webView = webView;
        webView.addJavascriptInterface(this,name);
        return this;
    }

    @Override
    public final  void callFunction(String fn, String... arguments)//封装了webview调用js的方法
    {
        if (webView == null)
        {
            return;
        }
        String functionStr = getFunctionStr(fn, arguments);
        webView.loadUrl(functionStr);

    }

    private String getFunctionStr(String fn, String... arguments)
    {
        StringBuilder stringBuilder = new StringBuilder("javascript:"+fn+"(");
        for (String argument : arguments)
        {
            stringBuilder.append(argument+",");
        }
        String s = stringBuilder.toString();
        String substring = s.substring(0, s.lastIndexOf(","));
        return substring+")";
    }
}
```
通过定义接口和进一步封装也方便你写一些单元测试来着，接下来就可以根据业务创建一些子类实现了
``` java
public class MainJavaScript extends BaseJavaScript implements JavaScript
{


    @Override
    public String getManufacturer()
    {
       //TODO
        return null;
    }

    @Override
    public String getVersionName()
    {
        //TODO
        return null;
    }
}
```
## 在MVP中的实践
我自己的项目中是只实现一个子类来统一管理所有的js接口，因为做的是混合应用，原生端主要提供一些本地服务给h5页面调用，这些服务接口都是一样的。最开始还没做MVP的时候是每个页面都给一个JS实现子类，改起来真的是很蛋疼，下面是现在项目中公共presenter中的一些类和接口，这边我是把Activity作为presentter的。

{% asset_img 1.webp %}
这边的BaseJS和上面的MainJavaScript是一样的
``` java
public class BaseJS extends JavascriptInteraction
{

    private final IActivity activity;

    public BaseJS(IActivity activity)
    {
        this.activity = activity;
    }

    @JavascriptInterface
    public String getManufacturer()
    {
        return Build.MANUFACTURER.toLowerCase().replaceAll("", "");
    }
    @JavascriptInterface
    public void scanQR()
    {
        activity.scanQR();
    }

}
```
IActivity接口中定义的方法大部分跟BaseJS是一样的，但这些方法是无法或者不好在BaseJS中实现的，这样就可以把实现延迟到具体的Acitivity中，而且又能统一管理这些js接口。
``` java
public interface IActivity
{

    void loginOK(String userInfo);

    void showWelcome(boolean isShow);
//...
}
```
然后在BaseJSActivity中对方法进行具体的实现了。
``` java
public abstract class BaseJSActivity extends BaseActivity implements IActivity
{

    protected static final String TAG = "BaseJSActivity";

    protected BaseJS js;

    protected User user;
    
     @Override
    public void loginOK(String userInfo)
    {
        User.isLogin = true;
        user = UserModel.getInstance(this).writeUser2SP(userInfo);
        checkLoginBeforeLoad(url);
    }
}
```
其他的Activity只要继承该基类就可以了，不同的业务逻辑只要重写部分代码就行了。如果后期js接口部分需要改动，维护扩展起来就挺方便的了。
## 总结
这篇文章主要是介绍一下自己管理webview与js的交互接口的一些经验，因为自己水平有限，仅供交流参考，如果文中有不妥和错误，欢迎指正。
