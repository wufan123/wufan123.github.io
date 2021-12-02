---
title: Android中反射的简单应用
date: 2016.05.31 13:06
categories: 
    - [Android]
tags: [Android]
---
自己对反射的理解和应用还处于比较浅显的阶段，写这篇文章更多在于整理总结，也就是帮助自己进一步的理解和学习反射机制。

## 反射
反射的概念是由Smith在1982年首次提出的，主要是指**程序可以访问、检测和修改它本身状态或行为的一种能力**。
## java中类反射
反射是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序对**自身进行检查，或者说“自审”，并能直接操作程序的内部属性和方法**。
简单总结这些定义，那就是反射可以让我们获得一个类的所有信息，包括私有属性和私有方法，对于我们这种小白，先知道这点就可以啦，那在java中如何使用发射呢。这里我们随便创建一个类来演示。比如说创建一个Book类：
``` java
public class Book implements Parcelable
{
    private int id=1;
    private String name="android";

    private String author="wf";

    private String getName()
    {
        return name;
    }
}
```
<!-- more -->
Book类中属性和方法都是私有的，现在我们通过反射来访问这些属性和方法。
``` java
String s = null;
try
{
    Class<?> bookClass = Class.forName("cc.abto.demo.Book");//完整类名
    Object book = bookClass.newInstance();//获得实例
    Method getAuthor = bookClass.getDeclaredMethod("getName");//获得私有方法
    getAuthor.setAccessible(true);//调用方法前，设置访问标志
    s = (String) getAuthor.invoke(book);//使用方法
}
catch (Exception e)
{
    e.printStackTrace();
}
```
可以看到上面代码中我们用Class和Method这两个类完成了反射，这两个类分别对应了类和方法，也就是包装了类和方法的信息，下面对反射的部分API做一下简单介绍：

 -  Class类：代表一个类，位于java.lang包下
 -  Field类：代表类的成员变量（成员变量也称为类的属性）
 -  Method类：代表类的方法
 -  Constructor类：代表类的构造方法
 -  Array类：提供了动态创建数组，以及访问数组的元素的静态方法

在Java中，每个class都有一个相应的Class对象。也就是说，当我们编写一个类，编译完成后，在生成的.class文件中，就会产生一个Class对象，用于表示这个类的类型信息。 java中的Class三种获取方式：
``` java
//使用Class类的静态方法forName()，用类的名字获取一个Class实例
Class<?> bookClass = Class.forName("cc.abto.demo.Book");

//利用对象调用getClass()方法获取该对象的Class实例
Book book = new Book();
Class<? extends Book> bookClass = book.getClass();

//运用.class的方式来获取Class实例，对于基本数据类型的封装类，还可以采用.TYPE来获取相对应的基本数据类型的Class实例
Class<Book> bookClass = Book.class;
Class<Integer> type = Integer.TYPE;
```
然后再贴一些常用的方法
``` java
    public Annotation[] getAnnotations () //获取这个类中所有注解

    getClassLoader() //获取加载这个类的类加载器

    getDeclaredMethods() //获取这个类中的所有方法

    getReturnType() //获取方法的返回类型

    getParameterTypes() //获取方法的传入参数类型

    isAnnotation() //测试这类是否是一个注解类

    getDeclaredConstructors() //获取所有的构造方法

    getDeclaredMethod(String name, Class… parameterTypes)// 获取指定的构造方法（参数：参数类型.class）

    getSuperclass() //获取这个类的父类

    getInterfaces()// 获取这个类实现的所有接口

    getFields() //获取这个类中所有被public修饰的成员变量

    getField(String name) //获取指定名字的被public修饰的成员变量

    newInstance() //返回此Class所表示的类，通过调用默认的（即无参数）构造函数创建的一个新实例
```
更多的方法和方法的注解大家可以查看文档。

## Android中的简单应用
查看Android SDK的源码时候。你会发现很多类或方法中经常加上了“@hide”注释标记，这些API是不允许在程序中调用的。Hidden API之所以被隐藏，是想阻止开发者使用SDK中那些未完成或不稳定的部分（接口或架构）。如图所示
{% asset_img 1.webp %}
所以在开发中，我们不仅可以通过反射获取私有属性和方法，也可以利用反射获取一些SDK对外部隐藏的API，比如说前阵子在做蓝牙开发的时候，自动配对的一些方法在API 19以后才对外开放的，这边我们就可以使用反射来实现配对功能了
``` java
try
{
    Class<BluetoothDevice> bluetoothDeviceClass = BluetoothDevice.class;
    bluetoothDeviceClass.getMethod("setPin", byte[].class).invoke(device, "1234".getBytes());
    bluetoothDeviceClass.getMethod("createBond").invoke(device);
    bluetoothDeviceClass.getMethod("setPairingConfirmation", boolean.class).invoke(device, true);
    bluetoothDeviceClass.getMethod("cancelPairingUserInput").invoke(device);

}
catch (Exception e)
{
    e.printStackTrace();
}
```
## 反射的好处
反射不仅可以让我们获得隐藏的方法和属性，还可以让**对象的实例化从编译时转化为运行时**，因为我们可以通过Class.forName("cc.abto.demo.Book").newInstance()的方法来生成新的实例，而这边的"cc.abto.demo.Book"是一个字符串，完全可以用变量来代替，再结合抽象工厂模式什么的，我们就可以很大程度上对程序应用中的功能模块进行解耦合。可能这边简单几句没能解释清楚，大家可以看看《大话设计模式》之类的书，里面就介绍的比较清楚明白了。
##反射的弊端
反射带来的两大弊端可能就是安全和性能问题了吧，这方面我知之甚少，有待进一步的了解和学习。
## 最后
因为自己水平有限，如果有些错误的地方还请大家见谅。下面贴出几篇写得比较好和详细的博客。
- [【Android】 认识反射机制（Reflection）](http://blog.qiji.tech/archives/4374)
- [Java反射机制的原理及在Android下的简单应用](http://www.cnblogs.com/crazypebble/archive/2011/04/13/2014582.html)
- [java中的反射机制](http://zlb1986.iteye.com/blog/937781)
- [Android注解与反射机制](http://efany.github.io/2016/04/02/Android%E6%B3%A8%E8%A7%A3%E4%B8%8E%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6/)
