四大组件是哪四个？ABCS(Activity,Braodcast,ContentProvider,Service)

**2.1 Activity**

1.Activity是什么？

2.典型情况下的Activity生命周期？

```
1.生命周期
onCreate()
onStart()
onResume()
onPause()
onStop()
onRestart()
onDestroy()

一些额外的生命周期
onConfigurationChanged()//横竖屏切换
onActivityResult()
onBackPressed()//按返回键
onNewIntent()//launch mode为singleInstance时
onSaveInstance()
onRestoreInstance()
```

3.异常情况下的Activity的生命周期 & 数据如何保存和恢复？

```
onSaveInstance() 和 onRestoreInstance()
```
4.从Activity A跳转到Activity B之后,然后再点击back键后，他们的生命周期调用流程是什么？

```
1.猜想
Activity A的生命周期调用流程：
onPause() -> onStop() .... onRestart() -> onStart() -> onResume()

Activity B的生命周期流程：
onCreate() -> onStart() -> onResume() -> onPause() -> onStop() ->onDestroy()
```

```
2.实际运行结果：
 MainActivity  --  onPause()
 ThirdActivity  --  onCreate()
 ThirdActivity  --  onStart()
 ThirdActivity  --  onResume()
 MainActivity  --  onStop()
 ThirdActivity  --  onPause()
 MainActivity  --  onRestart()
 MainActivity  --  onStart()
 MainActivity  --  onResume()
 ThirdActivity  --  onStop()
 ThirdActivity  --  onDestroy()
 
 3.结论：
 猜想与实际运行的结果是一致的。
```

5.如何统计Activity的工作时间？

```
1.在onResume时获取系统时间，在onStop时再次获取系统时间然后计算工作时间。
```


6.给我说说Activity的启动模式 & 使用场景。

```
可以参考以下的文章，只是简单分析manifest中配置的launch mode，不考虑启动activity时设置的flag

Android的四种LaunchMode：
http://note.youdao.com/noteshare?id=dee151fd534d6b15c71aea250b812f10&sub=WEB41a1ba84d4280ce8085eefed12e5a41b
```

7.如何在任意位置关掉应用所有Activity & 如何在任意位置关掉指定的Activity？

```
以下是我的答案，仅供参考：

1.首先可以用任意数据结构去将activity进行存储，待需要关闭的时候对activity进行移除finish；
2.获取activity栈的实例，然后对所有activity进行出栈操作。

1.如上述1所操作，用map将activity存起来，然后需要移除的时候获取activity对象finish调然后从map中remove掉
2.通过反射，去获取指定activity对象，然后通过invoke方法去调用activity的finsh方法。
```


8.Activity的启动流程(从源码角度解析)？

```
猜想：与activity service manager/ActivityThread/handler有关
```


9.启动一个其它应用的Activity的生命周期分析。
```

```


10.Activity任务栈是什么？在项目中有用到它吗？说给我听听

```
activity任务栈是一个管理activity的栈，遵循栈的先进后出的原则，activity的加载与销毁就是一个进栈出栈的过程

用途：实现一键出栈
```


11.什么情况下Activity不走onDestory?

```
1.点击home 键回到桌面的时候
2.点击返回键，回退到桌面的时候
3.activity失去焦点但有没有被销毁的时候
```


12.什么情况下Activity会单独执行onPause?

```

```


13.a->b->c界面，其中b是SingleInstance的，那么c界面点back返回a界面，为什么？

```
因为singleInstance启动模式时单独启动一个任务栈去存放activity，所以AC还是在同一个任务栈，当C回退时自然是返回A了。
```


14.如果一个Activity弹出一个Dialog,那么这个Acitvity会回调哪些生命周期函数呢？

```
自身弹出的dialog，activity不会回调任何生命周期函数
```

15.Activity之间如何通信 & Activity和Fragment之间通信 & Activity和Service之间通信？

```
1.activity之间通信：intent
2.activity语言fragment：intent
3.activity与service：intent
```


16.说说Activity横竖屏切换的生命周期。

```
文档：Android横竖屏切换时生命周期的变化.md
链接：http://note.youdao.com/noteshare?id=f3df384dd01f3f558299878989298564&sub=WEBda1fe30bf8c18a296957c4430f32efd8
```

17.前台切换到后台，然后在回到前台时Activity的生命周期。

```
onPause() -> onStop() -> onRestart() -> onStart() -> onResume()
```

18.下拉状态栏时Activity的生命周期？

```
activity不会回调任何生命周期函数
```

19.Activity与Fragment的生命周期比较？

20.了解哪些Activity常用的标记位Flags？

```
FLAG_NEW_TASK
```


21.谈谈隐式启动和显示启动Activity的方式？
```
显式启动activity：startactivity，通过intent去设置要跳转的activity

隐式启动activity：先在manifest中声明activity，定义intent-filter中的action 的name和scheme，然后intent跳转的时候设置action为对应的name进行跳转
```


22.Activity用Intent传递数据和Bundle传递数据的区别？为什么不用HashMap呢？

23.在隐式启动中Intent可以设置多个action,多个category吗 & 顺便讲讲它们的匹配规则？

24.Activity可以设置为对话框的形式吗？

```
可以
```


25.如何给Activity设置进入和退出的动画？

```
overridePendingTransition(int in,int out)方法，然后传入自定义的animation
```


26.Activity使用Intent传递数据是否有限制 & 如果传递一个复杂的对象，例如一个复杂的控件对象应该怎么做？

```
使用Intent传递数据是有限制的，大小限制 BINDER_VM_SIZE=（110241024）-（4096*2）也就是小于1M，类型限制
```


**2.2 BroadcastReceiver**

1.广播是什么？

2.广播的注册方式有哪些？

```
广播的注册方式有两种：一、静态注册（通过manifest的方式去生命） 二、动态注册，即通过代码注册
```


3.广播的分类 & 特性 & 使用场景？

```
广播主要有以下几类：
1)普通广播(Normal Broadcast)
2)系统广播(System Broadcast)
3)有序广播(Ordered Broadcast)
4)粘性广播(Sticky Broadcast) 于Android5.0 & API21已失效
5)本地广播(Local Broadcast)
```

4.说说系统广播和本地广播的原理 & 区别 & 使用场景。

5.有两个应用注册了一样的广播，一个是静态，一个是动态，连优先级也一样，那么当广播从系统发出来后，哪个应用先接收到广播？

**2.3 ContentProvider**

1.什么是内容提供者？

2.说说如何创建自己应用的内容提供者 & 使用场景。

3.说说ContentProvider的原理。

4.ContentProvider,ContentResolver,ContentObserver之间的关系？

5.说说ContentProvider的权限管理。

**2.4 Service**

1.什么是Service?

2.说说Service的生命周期。

```
说到service的生命周期，主要区分两种service的启动方式，不同的启动方式service的生命周期是不同的。
1.通过startService：
    onCreate()
    onStartCommand()
    nDestroy()

2.通过bindService
    onCreate()
    onBind()
    onUnbind()
    onDestroy()
```


3.Service和Thread的区别？

4.Android 5.0以上的隐式启动问题及其解决方案。

5.给我说说Service保活方案

6.IntentService是什么 & 原理 & 使用场景 & 和Service的区别。

7.创建一个独立进程的Service应该怎样做？

8.Service和Activity之间如何通信？

9.说说你了解的系统Service。

10.谈谈你对ActivityManagerService的理解。

11.在Activtiy中创建一个Thread和在一个Service中创建一个Thread的区别？

**2.5 Handler**

1.子线程一定不能更新UI吗？

```
不然运行时会报错误
```


2.给我说说Handler的原理

```
大概流程：通过sendMessage(Msg msg)/sendEmptyMEssage()方法将消息Message放入MessageQueue，MessageQueue是一个队列，采用先进先出的原则，然后通过Looper.loop()方法将消息从消息队列MessageQueue里面拿出来，然后Handler通过handlerMessage(Msg msg)方法处理传递过来的消息Message
```

3.Handler导致的内存泄露你是如何解决的？

4.如何使用Handler让子线程和子线程通信？

```
可以使用ThreadHandler
```

5.你能给我说说Handler的设计原理？

```
基于消息传递机制，通过死循环去处理消息
```

6.HandlerThread是什么 & 原理 & 使用场景？

```
HandlerThread 本质是一个线程，继承自Thread类，原理就是通过Thread去开启子线程，主要适用于子线程间的通信。
```

7.IdleHandler是什么？

8.一个线程能否创建多个Handler,Handler和Looper之间的对应关系？

9.为什么Android系统不建议子线程访问UI？

10.Looper死循环为什么不会导致应用卡死？

```
要想知道这个问题的答案，必须先理解Looper的运作原理
```


11.使用Handler的postDealy后消息队列有什么变化？

12.可以在子线程直接new一个Handler出来吗？

13.Message对象创建的方式有哪些 & 区别？

**2.6 AsyncTask**

1.AsyncTask是什么？能解决什么问题

```
AsyncTask是异步任务，主要用于实现异步回调的问题。我们在开发中，很多情况下都是异步执行的，所以异步任务提供了一种异步实现的操作。
```

2.给我谈谈AsyncTask的三个泛型参数作用 & 它的一些方法作用。

```
三个泛型参数分别为：Params, Progress, Result
```

3.给我说说AsyncTask的原理。

```
线程池
```

4.你觉得AsyncTask有不足之处吗？

**2.7 Fragment**

1.Android中v4包下Fragment和app包下Fragment的区别是什么？

2.Fragment的生命周期 & 请结合Activity的生命周期再一起说说。

3.说说Fragment如何进行懒加载。

4.ViewPager + Fragment结合使用会出现内存泄漏吗 & 如何解决？

5.Fragment如何和Activity进行通信 & Fragment之间如何进行通信？

6.给我谈谈Fragment3种切换的方式以及区别 & 使用场景。

7.getFragmentManager,getSupportFragmentManager,getChildFragmentManager之间的区别？

8.FragmentPagerAdapter和FragmentStatePagerAdapter区别？

9.Fragment如何实现类似Activity栈的压栈和出栈效果的？

**2.8 序列化**

1.什么是序列化 & 能用来干什么？

```
用于存储和传输
```


2.Android中序列化方式有几种？说说它们的区别。

```
目前是两种，一种是现实Serializable，另一种是实现Parcelable，并覆写对应方法
```


3.如果想要序列化的类中某些字段不序列化，那么应该怎么做？

**2.9 IPC**

1.说说你对Android多进程开发的认识？

2.Android中进程间通信的方式有哪些？

3.什么是AIDL?如何创建一个AIDL。

```
AIDL：android interface define language
```


**2.10 文件存储**

1.说说Android中数据持久化的方式 & 使用场景。

2.接触过MMKV吗？说说SharedPreference和它的区别。

3.第三方数据库框架用过哪些？有没有自己封装过一个SQLite的库？

```
GreenDao
```


4.SQLite是线程安全的吗 & SharedPreference是线程安全的吗？

5.请简单的给我说说什么是三级缓存？

6.SharedPreference的apply和commit的区别。

```
apply 和 commit的区别：

void apply()； apply是没有返回值的

boolean commit()； commit是带有布尔型的返回值

commit是支持异步的，而apply只适用于主线程的同步操作
```


7.谈谈你对SQLite事务的认识。

8.千奇百怪的SQL语句考察。

9.SharePreference跨进程使用会怎么样？如何保证跨进程使用安全？

10.谈谈SQLite升级要注意哪些地方？

**2.11 ListView & RecyclerView**

1.ListView是什么？如何使用？

2.RecyclerView是什么？如何使用？如何返回不一样的Item。

3.ListView和RecycyclerView的区别是什么？

4.分别讲讲你对ListView & RecyclerView的优化经验。

5.给我说说RecyclerView的回收复用机制

6.说说你是如何给ListView & RecyclerView加上拉刷新 & 下拉加载更多机制。

7.谈谈你是如何对ListView & RecycleView进行局部刷新的？

8.谈谈如何进行分页加载？

9.ScrollView下嵌套一个ListView通常会出现什么问题？

```
会发生活动冲突的情况。解决请参考
文档：事件分发机制解读及滑动冲突解决思路.m...
链接：http://note.youdao.com/noteshare?id=1404fcef25d0944c272ed9a3a90b29f4&sub=WEB75fe9eebf4582c28701e774a19cb1baf
```


10.一个ListView或者一个RecyclerView在显示新闻数据的时候，出现图片错位，可
能的原因有哪些 & 如何解决？

**2.12 图片编程**

1.你对Bitmap了解吗？它在内存中如何存在？

2.有关Bitmap导致OOM的原因知道吗？如何优化？

3.给我谈谈图片压缩。

4.LruCache & DiskLruCache原理。

```
文档：LruCache源码阅读及简单使用.md
链接：http://note.youdao.com/noteshare?id=be21830d31e20c3f2b591875ecabd16a&sub=5C0FEF06290E4CD18DFF3B713EFD55CE
```

5.说说你平常会使用的一些第三方图片加载库,最好给我谈谈它的原理。

```
Glide是我使用的最多的一个图片加载框架，功能十分强大。其次第三方图片架子啊框架还有 picasso（Square）和 fresco（Facebook的框架）以及 imageloader
```


6.如果让你设计一个图片加载库，你会如何设计？

7.有一张非常大的图片,你如何去加载这张大图片？

8.你知道Android中处理图片的一些库吗(OpenCv & GPUImage ...)？

9.如何计算一张图片在内存中占用的大小？

**2.13 WebView**

1.WebView是什么？
```
WebView是一个容器，主要用于加载H5页面
```

2.WebView会导致内存泄露吗？原因是什么？解决方式有哪些？

```
WebView是会导致内存泄漏的，
```


3.你知道Hybrid开发吗？说说你的相关经验。

4.说说WebSettings & WebViewClient & WebChromeClient这三个类的作用 & 
用法。

5.说说你了解的Hybrid框架。

**2.14 ViewPager**

1.什么是ViewPager?说说它的那些适配器。

2.你了解ViewPager2吗？和ViewPager 1有哪些区别？

3.ViewPager + Fragment结合使用存在的内存泄漏的原因是什么？如何解决？

**2.15 View事件分发机制**

1.什么是事件分发机制？主要用来解决什么问题？

```
主要解决用户的点击，触摸等事件
```


2.给我说说事件分发的流程 & 你项目解决事件冲突的一些案例。

```
文档：事件分发机制解读及滑动冲突解决思路.m...
链接：http://note.youdao.com/noteshare?id=1404fcef25d0944c272ed9a3a90b29f4&sub=WEB75fe9eebf4582c28701e774a19cb1baf
```

3.多点触摸事件平时接触过吗？如何监听用户第二个手指，第三个...？

4.OnTouchListener & OnTouchEvent & onClickListener三者之间的关系？

5.谈谈你对MotionEvent的认识？Cancel事件是什么情况下触发的？

6.能给我谈谈Android中坐标体系吗？

**2.16 View绘制机制**

1.说说View绘制流程。

```
onMeasure onlayout ondraw
```

2.说说Activity View树结构。

3.自定义View的方式有哪些?给我说说你之前项目中的案例。

4.invalidate和postvalidate的区别？

5.说说你在自定义View时常常重写的一些方法？

```
重写最多的就是onDraw()，其次是onMeasure()，最后是onLayout()
```

6.说说自定义View中如何自定义属性？

7.requestLayout(),onLayout(),onDraw(),drawChild()区别和联系？

8.如何计算出一个View的嵌套层级？

9.自定义View如何考虑机型适配？

**2.17 布局**

1.说说Android中有哪些布局 & 特点。

```
android中有：LinearLayout（线性布局）、RelativeLayout（相对布局）、FrameLayout（帧布局），GridLayout（网格布局），ConstainLayout（约束布局）
```


2.你知道布局文件到控件对象的过程吗？

3.有这么一个布局需求，一个文本控件放在屏幕一半的一半的中间位置，你如何进
行布局？

4.LinearLayout,FrameLayout,RelativeLayout性能对比，为什么？

**2.18 Binder**

1.什么是Binder？用来干什么？

2.给我具体讲讲Binder机制。

**2.19 动画机制**

1.Android中的动画分为哪些种类 & 特点 & 缺点。

2.知道SVG & 矢量动画吗？

3.给我说说转场动画。

4.给我谈谈插值器 & 估值器 的作用。

5.说说Android动画框架实现的原理。

**2.20 JNI**

1.什么是JNI?它主要用来干什么。

2.Java Native方法如何和Native函数进行绑定的？

3.JNI如何实现数据传递？

4.如何全局捕获Native发生的异常？

5.只有C/C++能编写Native库吗？

**2.21 Window & Appliction & Context**

1.说说你对Android中Window的理解。

2.说说你对Application的理解 & 生命周期。

3.Android中有哪些上下文 & 区别 & 作用。

4.谈谈你对Android中Context的理解。

**2.22 通知**

1.Android 8.0如何适配通知？

2.自定义通知流程？

**2.23 对话框(Dialog & DialogFragment & PopWindow)**

1.说说Android中对话框可以用哪些方式完成？

**2.24 蓝牙**

1.说说最新的蓝牙版本？新版本的特性是什么？

**2.25 冷启动&热启动**

1.什么是冷启动 & 什么是热启动 & 它们的流程？

2.如何优化冷启动？

3.启动页白屏，黑屏，太慢如何解决？

**2.26 悬浮窗**

1.在做悬浮窗的时候你遇到了什么困难(主要指悬浮窗权限适配)？

2.如何制作一个悬浮窗？

**2.27 Android版本**

1.最新的Android版本多少知道吗？有哪些特性

2.说说更新较大的Android版本。

**2.28 Android Studio**

1.你现在比较常用Android Studio那个版本 & 用的Gradle版本是多少？

```
android studio 3.3 / gradle 4.10.1 / android plugin version 3.3.0/2.3.3
```

2.如何理解gradle?

3.说说Android Studio中大致项目结构？ 

4.混淆是什么 & 为什么需要进行混淆 & 混淆的原理 & 
为什么Java反射常常会和混淆冲突？

**2.29 UI卡顿优化**

1.ANR是什么？导致原因有哪些？
2.谈谈你项目中避免ANR的一些经验。
3.分别说说Activity & BroadcastReceiver & Serice最长可耗时时间为多少？

```
文档：造成ANR的原因以及相关的解决办法.md
链接：http://note.youdao.com/noteshare?id=2e94fae5588c62c27a2af98bee325fa0&sub=WEB0c8e7cb3c8207a43111617955dbfb47f
```

**2.30 内存优化**

1.什么是OOM & 什么是内存泄漏 & 什么是内存抖动？

2.谈谈你项目中内存优化的一些经验。

**2.31 屏幕适配**

1.说说Android中一些屏幕单位。

```
dp，px，sp
```


2.谈谈你项目中的一些屏幕适配的经验。

```
少使用具体的宽高，尽可能的使用 match_parent和wrap_content，LinearLayout可以使用weight等
```


3.今日头条的轻量级适配方案了解吗 & 给我说说原理。

**2.32 多渠道打包 & apk签名**

1.apk为什么需要签名？

```
签名是一个apk的标识，手机安装apk的时候都会校验签名的有效性。
```

2.多渠道打包是什么 & 有类似经验吗？

3.简述多渠道打包及原理和常用操作？

**2.33 项目架构**

1.说说你用过的项目架构？

2.分别给我说说MVC,MVP,MVVM特点和区别。

3.以登陆界面为例子,设计MVP架构。

4.谈谈AndroidManifest.xml文件的理解。

**2.34 Android前沿知识**

1.谷歌新出的Flutter知道吗？

```
知道，使用flutter进行过界面开发。编程风格像web的js和react，vue
```


2.谷歌新出的官方开发语言Kotlin了解吗 & 和Java相比它有哪些特点。

```
了解相关的语法。 kotlin具有许多高级特性，如：协程，函数式编程，扩展函数，判空处理等。写界面的时候，能直接使用xml里面的id名称，代码更简洁了。kotlin的接口可以进行实现，而java不能。kotlin必须要使用open子类才能继承，更贴切开闭原则。
```


3.谈谈Kotlin中协程的认识？

**2.35 音视频开发(高薪)**

1.之前有过音视频开发经验吗 & 说说用哪些开源架子开发的。

2.FFmpeng了解过吗？

3.Android中播放视频音频的方式有哪些？

4.Android中播放网络地址视频有哪些出色的开源库？

5.流媒体服务器了解吗？

6.谈谈你对编码格式的理解。

7.MediaPlayer和SoundPool的区别？

8.视频硬解码和软解码的区别？

**2.36 其它Android部分有关面试题**

1.说说一个app的启动流程(从源码角度讲解)。

2.你知道无论是Kotlin或者是Java,程序运行的主要入口都是main()方法，那么Andr
oid的main方法在哪里？

```
在ActivityThread这个类里面
```


3.Android Hook技术了解吗？

4.简述Android中的加固和使用平台？

5.谈谈你对Apk瘦身的经验？

```
文档：Apk的体积优化.md
链接：http://note.youdao.com/noteshare?id=310b5a0a12e6a177801ab8f4a822d06f&sub=WEB0d9e2cba8b68d4747837c3907f356300
```


6.为什么子线程不能更新UI？

7.你知道如何定位内存泄漏吗？

8.说说System.exit(0),onDestory(),Activity.finish()的区别？

9.在OnResume或者之前获取View的宽高为多少 & 为什么？

10.Art & Dvm 区别，特别是谈谈GC的区别。

11.说说你用的二维码框架 & 有过优化经验吗？

12.谈谈App多进程的好处 & 缺点。

13.说说AMS是怎么找到启动指定的Activity？

14.View的getWidth和getMeasureWidth有啥区别？

15.有插件化或者热修复经验吗？说说它的原理。

16.断点续传了解吗？谈谈你是如何通过多线程实现断点续传的。

17.给我谈谈你对SurfaceView的认识。

18.什么情况下你会使用到ScrollView。

```
ScrollView使用场景：需要显示的数据超出了屏幕的宽高
```


19.低版本SDK如何使用高版本API？

20.AlertDialog,PopWindow,Activity之间的区别？

21.Application和Activity,Context的区别？

```
从最直观的角度，application：应用，activity：活动，context：上下文。
```


22.谈谈Android中多线程通信方式？

23.说说Android大体的架构图，试着画出来。

24.知道SpareArray吗？

25.Activity除了setContentView可以设置布局，还有其它方式吗？

26.Android为每个应用程序分配的内存大小为多少？

27.Android进程保活方案？

28.谈谈Android系统安装apk的过程？

29.Activity,Window,View三者的关系？

30.ActivityThread,ActivityManagerService,WindowManagerService的工作原理？

31.PackageManagerService的工作原理？

32.PowerManagerService的工作原理？

33.在桌面点击一个未启动的App的流程 & 点击一个已启动的App的流程？

34.Android中进程分为哪些种类？

35.什么是埋点，懂点它的原理吗？

```
埋点一般用于数据的收集，去记录用户的行为。如登录、注册埋点、付费埋点等。
原理就是调用数据接口，在对应的时机进行数据上报。
```


36.进程和Application生命周期之间的关系？

37.App相互唤醒的有哪些方式？

38.Android中如何开启多进程？应用是否可以开启N个进程？

39.谈谈消息推送的方式有哪些？

40.谈谈你对Root权限的理解。

41.谈谈项目如何进行国际化？

42.谈谈你对Intent和IntentFilter的理解。

43.一条最长的短信息约占多少byte？
