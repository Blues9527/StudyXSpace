# 四大组件
###### Activity
    1.生命周期
    onCreate()
    onStart()
    onResume()
    onPause()
    onStop()
    onRestart()
    onDestroy()
    
    一些额外的生命周期
    onConfigurationChanged()
    onActivityResult()
    onBackPressed()
    onNewIntent()
    onSaveInstance()
    onRestoreInstance()
    
###### Service
    1.service的两种启动方式
        1)startService
        生命周期为：
            onCreate()
            onStartCommand()
            onDestroy()
        
        2)bindService
        生命周期为:
            onCreate()
            onBind()
            onUnbind()
            onDestroy()
        
###### Broadcast Receiver
    1.广播接收注册的两种方式
        1)静态注册
        manifest.xml里通过<receive>标签注册
        
        2)动态注册
        context.registerReceiver()
        
    2.广播的类型
        1)普通广播(Normal Broadcast)
        2)系统广播(System Broadcast)
        3)有序广播(Ordered Broadcast)
        4)粘性广播(Sticky Broadcast)
            于Android5.0 & API21已失效
        5)本地广播(Local Broadcast)

###### Content Provider

###### Fragment
    1.生命周期
        onAttach()
        onCreate()
        onCreateView()
        onActivityCreated()
        onStart()
        onResume()
        onPause()
        onStop()
        onDestroyView()
        onDestroy()
        onDetach()
    2.

# Activity的启动过程
[转自@罗升阳](https://blog.csdn.net/luoshengyang/article/details/6689748)

- Step 1. Launcher.startActivitySafely
- Step 2. Activity.startActivity
- Step 3. Activity.startActivityForResult
- Step 4. Instrumentation.execStartActivity
- Step 5. ActivityManagerProxy.startActivity
- Step 6. ActivityManagerService.startActivity
- Step 7. ActivityStack.startActivityMayWait
- Step 8. ActivityStack.startActivityLocked
- Step 9. ActivityStack.startActivityUncheckedLocked
- Step 10. Activity.resumeTopActivityLocked
- Step 11. ActivityStack.startPausingLocked
- Step 12. ApplicationThreadProxy.schedulePauseActivity
- Step 13. ApplicationThread.schedulePauseActivity
- Step 14. ActivityThread.queueOrSendMessage
- Step 15. H.handleMessage
- Step 16. ActivityThread.handlePauseActivity
- Step 17. ActivityManagerProxy.activityPaused
- Step 18. ActivityManagerService.activityPaused
- Step 19. ActivityStack.activityPaused
- Step 20. ActivityStack.completePauseLocked
- Step 21. ActivityStack.resumeTopActivityLokced
- tep 22. ActivityStack.startSpecificActivityLocked
- Step 23. ActivityManagerService.startProcessLocked
- Step 24. ActivityThread.main
- Step 25. ActivityManagerProxy.attachApplication
- Step 26. ActivityManagerService.attachApplication
- Step 27. ActivityManagerService.attachApplicationLocked
- Step 28. ActivityStack.realStartActivityLocked
- Step 29. ApplicationThreadProxy.scheduleLaunchActivity
- Step 30. ApplicationThread.scheduleLaunchActivity
- Step 31. ActivityThread.queueOrSendMessage
- Step 32. H.handleMessage
- Step 33. ActivityThread.handleLaunchActivity
- Step 34. ActivityThread.performLaunchActivity
- Step 35. MainActivity.onCreate

> - 一. Step1 - Step 11：Launcher通过Binder进程间通信机制通知ActivityManagerService，它要启动一个Activity；
> - 二. Step 12 - Step 16：ActivityManagerService通过Binder进程间通信机制通知Launcher进入Paused状态；
> - 三. Step 17 - Step 24：Launcher通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就创建一个新的进程，用来启动一个ActivityThread实例，即将要启动的Activity就是在这个ActivityThread实例中运行；
> - 四. Step 25 - Step 27：ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给ActivityManagerService，以便以后ActivityManagerService能够通过这个Binder对象和它进行通信；
> - 五. Step 28 - Step 35：ActivityManagerService通过Binder进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。

# ANR(Application No Responding)产生的主要原因

    1.Activity:5s内无法响应用户输入事件(例如键盘输入, 触摸屏幕等)
    2.Broadcast Receiver:的事件(onRecieve方法)在规定时间内没处理完(前台广播为10s，后台广播为60s)
    3.service:前台20s后台200s未执行完成 Timeout executing service（Android O多了个startForeground 5s限制）
    
定位ANR：traces.txt

# Handler的消息传递机制
    
    用途：子线程发送数据，主线程接收数据，可用于UI更新，消息传递等
    涉及到的组件： Handler Looper Message MessageQueue
    方法：Looper.prepare() Looper.loop() handler.post() handler.sendMessage()
        handlerMessage()
    //Looper.getMainLooper()/Looper.myLooper() 获取循环器的对象
    
    大概流程：在子线程实例化一个Handler对象，然后通过sendMessage(Msg msg)/sendEmptyMEssage()方法将消息Message放入MessageQueue，MessageQueue是一个队列，采用先进先出的原则，然后在主线程通过Looper.prepare()方法初始化一个Looper循环器，然后通过Looper.loop()方法将消息从消息队列MessageQueue里面拿出来，然后Handler通过handleMessage(Msg msg)方法处理子线程传递过来的消息Message。
    
# view的绘制流程

[参考@Android_韦鲁斯](https://blog.csdn.net/sinat_27154507/article/details/79748010)

概述：
> 当一个应用启动的时候，会启动一个主Activity，Android系统会根据Activity的布局来对它进行绘制。绘制会从根视图ViewRootImpl的performTraversals()方法开始，从上到下遍历整个视图树，每一个View控件负责绘制自己，而ViewGroup还需要负责通知自己的子View进行绘制操作。
--------------------- 

View的绘制流程分为三个阶段，分别是measure、layout、draw;

- measure ：根据父 view 传递的 MeasureSpec 进行计算大小。
- layout ：根据measure子View所得到的布局大小和布局参数，将子View放在合适的位置上。
- draw ：把 View 对象绘制到屏幕上。




# android线程池实现原理
    
    
# android屏幕适配
[参考@JeanBoydev](https://blog.csdn.net/freekiteyu/article/details/70790711)
    
# android中的三级缓存
[参考@wanbo_](https://www.jianshu.com/p/2cd59a79ed4a)
- 网络缓存, 不优先加载, 速度慢,浪费流量
- 本地缓存, 次优先加载, 速度快
- 内存缓存, 优先加载, 速度最快

原理:
- 首次加载 Android App 时，肯定要通过网络交互来获取图片，之后我们可以将图片保存至本地SD卡和内存中，之后运行 App 时，优先访问内存中的图片缓存，若内存中没有，则加载本地SD卡中的图片。总之，只在初次访问新内容时，才通过网络获取图片资源。
 
### LruCache（LRU即Least Recently Used）源码解读
[可参考](https://www.jianshu.com/p/b6e9a7bdf8d3)
底层实现是一个LinkedHashMap<K,V>