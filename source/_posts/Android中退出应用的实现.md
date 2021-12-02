---
title: Android中退出应用的实现
date: 2016.05.30 18:03
categories: 
    - [Android]
tags: [Android]
---

退出应用是项目开发中很基本的一个需求，这个功能很简单，也有很多实现的方式，这里把自己知道的退出方法做一个整理，跟大家交流分享一下。
<!-- more -->
## 容器式
就是用一个全局容器把所有的Activity存起来，退出时遍历调用容器里Activity的finish方法就可以啦，这个效果很好，也很好操作的。创建一个单例来管理Acitivity，一般是在Application中实现的。
``` java
public class MyApplication extends Application
{

    private List<Activity> activityList = new LinkedList<Activity>();//LinkedList便于删除和增加

    // 省略了单例代码...

    public void removeActivity(Activity activity)
    {
        activityList.remove(activity);
    }

    public void addActivity(Activity activity)
    {

        activityList.add(activity);
    }

    public void exit()
    {

        for (Activity activity : activityList)
        {
            activity.finish();
        }
        System.exit(0);
    }
}
```
当当当，这样就OK了。开发中在BaseActivity这样的超类统一实现后，再在需要退出的地方调用exit方法就可以退出应用了。
``` java 
public abstract class BaseActivity extends AppCompatActivity
{

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        MyApplication.getInstance().addActivity(this);
    }
  //...
    @Override
    protected void onDestroy()
    {
        super.onDestroy();
        MyApplication.getInstance().removeActivity(this);
    }
 //...
}
```
## 广播式
在BaseActivity这样超类中注册一个广播，要退出时发送一个广播，就可以结束所有页面了。
``` java
public abstract class BaseActivity extends AppCompatActivity
{

    private static final String EXITACTION = "action.exit";

    private ExitReceiver exitReceiver = new ExitReceiver();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        IntentFilter filter = new IntentFilter();
        filter.addAction(EXITACTION);
        LocalBroadcastManager.getInstance(this).registerReceiver(exitReceiver, new IntentFilter(EXITACTION));
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        LocalBroadcastManager.getInstance(this).unregisterReceiver(exitReceiver);
    }

    class ExitReceiver extends BroadcastReceiver
    {
        @Override
        public void onReceive(Context context, Intent intent) {
            BaseActivity.this.finish();
        }
    }
}
```
发送广播的代码就不贴啦，上面的代码用的是本地广播，效率和安全性高一点。
## 标志式
就是在工具类或者配置类里定义一个静态的成员属性
``` java
public class Config
{
    public static final Boolean isExit =false;
//...
}
```
然后在BaseActivity的onResume方法中做判断就可以了。
``` java
public abstract class BaseActivity extends AppCompatActivity
{
  @Override
  onResume() {
     super.onResume();
     if(Config.isExit) finish();
  }
//...
}
```
## Intent.FLAG式
在startActivity的时候，可以在intent附加一些flag信息来控制Acitivity的启动模式，里面呢有一个FLAG_ACTIVITY_CLEAR_TASK的flag，根据文档的说明，它在启动目标Acitivity的时候会清空之前所有任务关联的Acitivity，嘿嘿，这不就我们想要的效果么，所以我们完全可以利用这个启动模式来做退出功能。我们只要建立一个ExitAcitivity。
``` java
public class ExitAcitivity extends AppCompatActivity
{

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        finish();
    }
}
```
然后呢，要退出的时候，通过前面提到的FLAG_ACTIVITY_CLEAR_TASK来启动这个Activity就可以啦。
``` java
public void onClick(View v)
{
    Intent intent = new Intent(this, ExitAcitivity .class);
    intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    startActivity(intent);
}
```
文档中说明FLAG_ACTIVITY_CLEAR_TASK要与FLAG_ACTIVITY_NEW_TASK一起使用，应该是底层实现的时候就直接把之前的Task给杀掉了。通过这种方式我们也可以简单的实现退出应用的功能，当然利用Activity的singleTask启动模式也是可以的哦。
## 总结
其实我一直觉得思维才是一个人的能力体现（啦啦啦，不装逼我们还是好朋友），虽然这只是一个简单的例子，但是很多时候，有些问题，你只要换一个角度去思考就迎刃而解了。所以写这些东西也是为了跟大家交流然后扩展自己的见解，那么，大家还有其他方法么，欢迎指导交流哦。
