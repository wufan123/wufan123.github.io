---
title: Android中AIDL的使用详解
date: 2016.06.09 16:46
categories: 
    - [Android]
tags: [Android]
cover: /images/android_banner.webp
---

为了说的深入浅出一点，我们先从AIDL的作用和使用说起，然后再开始介绍一些概念和工作原理。
## AIDL用来做什么
AIDL是Android中**IPC（Inter-Process Communication）**方式中的一种，AIDL是**Android Interface definition language**的缩写，对于小白来说，AIDL的作用是让你可以在自己的APP里绑定一个其他APP的service，这样你的APP可以和其他APP交互。
 <!-- more -->
## AIDL的使用
在android studio 2.0里面使用AIDL，因为是两个APP交互么，所以当然要两个APP啦，我们在第一个工程目录右键

{% asset_img 1.webp %}

输入名称后，sutido就帮我们创建了一个AIDL文件。
``` java
// IMyAidlInterface.aidl
package cc.abto.demo;

// Declare any non-default types here with import statements

interface IMyAidlInterface {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```
上面就是studio帮我生成的aidl文件。basicTypes这个方法可以无视，看注解知道这个方法只是告诉你在AIDL中你可以使用的基本类型（int, long, boolean, float, double, String），因为这里是要跨进程通讯的，所以不是随便你自己定义的一个类型就可以在AIDL使用的，这些后面会说。我们在AIDL文件中定义一个我们要提供给第二个APP使用的接口。
``` java
interface IMyAidlInterface {
   String getName();
}
```
定义好之后，就可以**sync project**一下，然后新建一个service。在service里面创建一个内部类，继承你刚才创建的AIDL的名称里的Stub类,并实现接口方法,在onBind返回内部类的实例。
``` java
public class MyService extends Service
{

    public MyService()
    {

    }

    @Override
    public IBinder onBind(Intent intent)
    {
        return new MyBinder();
    }

    class MyBinder extends IMyAidlInterface.Stub
    {

        @Override
        public String getName() throws RemoteException
        {
            return "test";
        }
    }
}
```
接下来，将我们的AIDL文件拷贝到第二个项目，然后**sync project**一下工程。

{% asset_img 2.webp %}

这边的包名要跟第一个项目的一样哦，这之后在Activity中绑定服务。
``` java
public class MainActivity extends AppCompatActivity
{


    private IMyAidlInterface iMyAidlInterface;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService(new Intent("cc.abto.server"), new ServiceConnection()
        {

            @Override
            public void onServiceConnected(ComponentName name, IBinder service)
            {

                iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
            }

            @Override
            public void onServiceDisconnected(ComponentName name)
            {

            }
        }, BIND_AUTO_CREATE);
    }

    public void onClick(View view)
    {
        try
        {
            Toast.makeText(MainActivity.this, iMyAidlInterface.getName(), Toast.LENGTH_SHORT).show();
        }
        catch (RemoteException e)
        {
            e.printStackTrace();
        }
    }
}
```
这边我们通过隐式意图来绑定service，在onServiceConnected方法中通过**IMyAidlInterface.Stub.asInterface(service)**获取iMyAidlInterface对象，然后在onClick中调用iMyAidlInterface.getName()。

{% asset_img 3.webp %}

## 自定义类型
如果我要在AIDL中使用自定义的类型，要怎么做呢。首先我们的自定义类型要实现**Parcelable**接口，下面的代码中创建了一个User类并实现Parcelable接口。这边就不对Parcelable进行介绍了，不熟悉的童鞋自行查找资料，总之我们这边可以借助studio的Show Intention Action（也就是Eclipse中的Quick Fix，默认是alt+enter键）帮我们快速实现Parcelable接口。

{% asset_img 4.webp %}

接下新建一个aidl文件，名称为我们自定义类型的名称，这边是User.aidl。在User.aidl申明我们的自定义类型和它的完整包名，注意这边parcelable是小写的，不是Parcelable接口，一个自定类型需要一个这样同名的AIDL文件。
``` java
package cc.abto.demo;
parcelable User;
```
然后再在我们的AIDL接口中导入我们的AIDL类型。
{% asset_img 5.webp %}
然后定义接口方法，**sync project**后就可以在service中做具体实现了。
``` java
public class MyService extends Service
{
    //...
    @Override
    public IBinder onBind(Intent intent)
    {
        return new MyBinder();
    }

    class MyBinder extends IMyAidlInterface.Stub
    {
        //...
        @Override
        public User getUserName() throws RemoteException
        {
            return new User("wswf");
        }
    }
}
```
最后将我们的AIDL文件和自定义类型的java一并拷贝到第二个项目，注意包名都要一样哦
{% asset_img 6.webp %}
然后就可以在Activity中使用该自定义类型的AIDL接口了
``` java
public class MainActivity extends AppCompatActivity
{
    //...
    public void onClick(View view)
    {
        try
        {
            Toast.makeText(MainActivity.this, iMyAidlInterface.getUserName().getName(), Toast.LENGTH_SHORT).show();
        }
        catch (RemoteException e)
        {
            e.printStackTrace();
        }
    }
}
```
效果图就不贴了哈，通过这种方式我们就可以让两个APP之间进行交互了。
## 最后
为什么APP间的进程交互这么麻烦，是因为它们属于不同的进程，之间的交互涉及到进程间的通讯。而AIDL只是Android中众多进程间通讯方式中的一种方式，那么AIDL到底是什么鬼，它是如何工作的，Android的IPC机制又是怎样的呢。我将在下一篇文章[Android中AIDL的工作原理](http://www.jianshu.com/writer#/notebooks/4326108/notes/4350318)中介绍。
