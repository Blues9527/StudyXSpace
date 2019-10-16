## 3.1 Android

#### Activity

Q：说下Activity的生命周期？
> A: onCreate(),onStart(),onResume(),onPause(),onStop(),onRestart(),onDestroy()

Q：onStart()和onResume()/onPause()和onStop()的区别？

Q：Activity A启动另一个Activity B会回调哪些方法？如果Activity B是完全透明呢？如果启动的是一个Dialog呢？

Q：谈谈onSaveInstanceState()方法？何时会调用？
> A:关闭屏幕，点解Home键回到桌面，打开任务界面等都会调用此方法

Q：onSaveInstanceState()与onPause()的区别？
> A:从方法来看，onSaveInstanceState方法中是携带参数，即在此声明周期下是能进行参数保存，避免activity被回收后重启时参数丢失的问题。

> 从功能来看，会执行onSaveInstanceState()方法的一定会先执行onPause()方法，而执行onPause()方法不一定会执行onSaveInstanceState()方法。比如说：点击回退按钮的时候，会直接调用onPause()和onDestroy()而不会调用onSaveInstanceState()

Q：如何避免配置改变时Activity重建？
> A:如果是横竖屏配置改变导致Activity重建，这个问题比较好解决，在manifest中在指定的activity中配置 

```
android:configChanges="orientation|keyboardHidden|screenSize"
```


Q：优先级低的Activity在内存不足被回收后怎样做可以恢复到销毁前状态？

Q：说下Activity的四种启动模式？（有时会出个实际问题来分析返回栈中Activity的情况）
> A:standrad/singleTop/singleTask/singleInstance

Q：谈谈singleTop和singleTask的区别以及应用场景
> A:singleTop:如果activity栈顶存在对应activity的实例，则复用，没有则在栈顶中创建一个对应activity的实例。

> singleTask:如果activity栈堆中存在对应activity的实例，则移除该activity前所有其他activity的实例，使该activity的实例成为栈顶。如果没有则在栈顶中创建一个实例。

> 应用场景：都可以防止用户快速点击而生成多个目标activity实例；singleTask：闪屏页

Q：onNewIntent()调用时机？
> A:当activity生命为singleInstance的时候会调用。首次跳转到目标activity后，再次跳转到目标activity时则会调用onNewIntent()

Q：了解哪些Activity启动模式的标记位？
> A:FLAG_ACTIVITY_SINGLE_TOP/FLAG_ACTIVITY_NEW_TASK/FLAG_ACTIVITY_CLEAR_TOP/FLAG_ACTIVITY_NO_HISTORY/FLAG_ACTIVITY_CLEAR_TASK

Q：如何启动其他应用的Activity？
> A:设置taskAffinity和scheme/通过ComponentName跳转 

Q：Activity的启动过程？

#### Fragment

Q：谈一谈Fragment的生命周期？
> A: onAttch()/onCreate/onCreateView()/onActivityCreated()/onStart()/onResume()/onPasue()/onStop()/onDestroyView()/onDestroy/onDetach()

Q：Activity和Fragment的异同？
> A:Activity是一个独立的界面，而fragment时一个碎片化界面，需要依赖在activity之上

Q：Activity和Fragment的关系？

Q：何时会考虑使用Fragment？
> A:比如说带有导航栏功能的，根据导航栏上的内容显示不同的界面，此时会优先考虑fragment

#### Service

Q：谈一谈Service的生命周期？
> A:onCreate()/onStartCommond()/onDestroy()

> A: onCreate()/onBind()/onUnbind()/onDestroy()

Q：Service的两种启动方式？区别在哪？
> A:service有两种启动方式，一种是通过startService(),另外一种是bindService()。通过bindService()方法启动service，此时service会绑定activity的声明周期，而通过startService()方法启动service，如果不主动去停止service的话，会一直运行下去。

Q：一个Activty先start一个Service后，再bind时会回调什么方法？此时如何做才能回调Service的destory()方法？
> A:onStart()和onBind()。停止需要调用unbindService()和stopService()

Q：Service如何和Activity进行通信？

Q：用过哪些系统Service？

Q：是否能在Service进行耗时操作？如果非要可以怎么做？

Q：AlarmManager能实现定时的原理？

Q：前台服务是什么？和普通服务的不同？如何去开启一个前台服务？

Q：是否了解ActivityManagerService，谈谈它发挥什么作用？

Q：如何保证Service不被杀死？

#### Broadcast Receiver

Q：广播有几种形式？什么特点？
> A:普通广播/本地广播/粘性广播/系统广播/有序广播

Q：广播的两种注册形式？区别在哪？
> A:主要分为静态注册和动态注册，静态注册是在manifest中进行声明，动态注册是在Java代码中进行声明，context.registerBroadcast()

#### ContentProvider

Q：ContentProvider了解多少？

#### 数据存储

Q：Android中提供哪些数据持久存储的方法？
> sp,content provider,sqlite

Q：Java中的I/O流读写怎么做？

Q：SharePreferences适用情形？使用中需要注意什么？

Q：了解SQLite中的事务处理吗？是如何做的？

Q：使用SQLite做批量操作有什么好的方法吗？

Q：如果现在要删除SQLite中表的一个字段如何做？

Q：使用SQLite时会有哪些优化操作?

#### IPC

Q：Android中进程和线程的关系？区别？

Q：为何需要进行IPC？多进程通信可能会出现什么问题？

Q：什么是序列化？Serializable接口和Parcelable接口的区别？为何推荐使用后者？
> A:在数据进行传递的时候，需要将数据进行序列化后才能传输。就是将数据转换成而进行的流，反序列化就是将流转换成对象。

> A: parcelable的效率比serializable高，而且parcelable也是android专门实现序列化的一个接口。

Q：Android中为何新增Binder来作为主要的IPC方式？

Q：使用Binder进行数据传输的具体过程？

Q：Binder框架中ServiceManager的作用？

Q：Android中有哪些基于Binder的IPC方式？简单对比下？

Q：是否了解AIDL？原理是什么？如何优化多模块都使用AIDL的情况？

#### View

Q：MotionEvent是什么？包含几种事件？什么条件下会产生？
> A:ACTION_DOWN/ACTION_UP/ACTION_MOVE/ACTION_CANCEL

Q：scrollTo()和scrollBy()的区别？
> A:scrollBy内部调用的还是scrollTo

Q：Scroller中最重要的两个方法是什么？主要目的是？
>

Q：谈一谈View的事件分发机制？

Q：如何解决View的滑动冲突？
> 不同向:判断所属区域，判断滑动的方向，然后进行对应的拦截

> 同向:判断所属区域，然后进行对应的拦截

Q：谈一谈View的工作原理？
> 测量(onMeasure) -> 布局(onLayout) -> 绘制(onDraw)

Q：MeasureSpec是什么？有什么作用？

Q：自定义View/ViewGroup需要注意什么？

Q：onTouch()、onTouchEvent()和onClick()关系？

Q：SurfaceView和View的区别？

Q：invalidate()和postInvalidate()的区别？

#### Drawable等资源

Q：了解哪些Drawable？适用场景？

Q：mipmap系列中xxxhdpi、xxhdpi、xhdpi、hdpi、mdpi和ldpi存在怎样的关系？

Q：dp、dpi、px的区别？

Q：res目录和assets目录的区别？

#### Animation

动画分为：视图动画与属性动画，两者的区别为：view的位置是否真正改变了。而视图动画有包括：补间动画与帧动画。

Q：Android中有哪几种类型的动画？
> A:属性、补间、帧动画

Q：帧动画在使用时需要注意什么？

Q：View动画和属性动画的区别？
> A:view的位置是否真正改变

Q：View动画为何不能真正改变View的位置？而属性动画为何可以？
> A:帧动画是通过一系列图片拼凑成的，通过一张张图片改变来产生动画的效果，实际的view是没有改变的，改变的只是图片。

> A:补间动画是通过将图片旋转、平移、缩放、改变透明度等操作来实现动画效果的，但是view始终是不变的。

> A:而属性动画，是通过改变view的属性去实现动画的效果。

Q：属性动画插值器和估值器的作用？

#### Window

Q：Activity、View、Window三者之间的关系？

Q：Window有哪几种类型？

Q：Activity创建和Dialog创建过程的异同？

#### Handler

Q：谈谈消息机制Handler？作用？有哪些要素？流程是怎样的？
> A: handler消息机制主要包括的组件：Handler、Looper、MessageQueue、Message

> A:Handler消息机制主要实现线程间的通信，主要的流程是：
在A线程中调用sendMessage和sendEmptyMessage进行发送消息，然后B线程的MessageQueue调用enqueue方法将Message压入MessageQueue，而MessageQueue采用的是队列的方式，采用的是先进先出策略，B线程的Looper器调用loop方法不断地从MessageQueue中拿Message出来给B线程的Handler处理，Handler是通过obtainMessage方法从Looper中获取消息的，通过handleMessage方法进行处理。

Q：为什么系统不建议在子线程访问UI？
> A:在android中UI线程也称主线程，直接在子线程对UI进行操作会报出异常提示：请在UI线程进行UI操作。防止多个线程同时更新UI。

Q：一个Thread可以有几个Looper？几个Handler？
> A:1个Looper和多个Handler

Q：如何将一个Thread线程变成Looper线程？Looper线程有哪些特点？

Q：可以在子线程直接new一个Handler吗？那该怎么做？
> A:可以，在Handler的构造函数中传入指定的Looper

Q：Message可以如何创建？哪种效果更好，为什么？

Q：这里的ThreadLocal有什么作用？
> 存放Looper的

Q：主线程中Looper的轮询死循环为何没有阻塞主线程？
> A:因为android是基于事件（消息）驱动的，android能实现响应各种事件的原理就是通过Looper死循环去轮询获取用户的操作，然后交给Handler去进行处理各种事件。


Q：使用Hanlder的postDealy()后消息队列会发生什么变化？

#### 线程

Q：Android中还了解哪些方便线程切换的类？

Q：AsyncTask相比Handler有什么优点？不足呢？

Q：使用AsyncTask需要注意什么？

Q：AsyncTask中使用的线程池大小？

Q：HandlerThread有什么特点？

Q：快速实现子线程使用Handler

Q：IntentService的特点？

Q：为何不用bindService方式创建IntentService？

Q：线程池的好处、原理、类型？

Q：ThreadPoolExecutor的工作策略？

Q：什么是ANR？什么情况会出现ANR？如何避免？在不看代码的情况下如何快速定位出现ANR问题所在？

#### Bitmap

Q：加载图片的时候需要注意什么？

Q：LRU算法的原理？

Q：Android中缓存更新策略？

#### 性能优化

Q：项目中如何做性能优化的？

Q：了解哪些性能优化的工具？

Q：布局上如何优化？列表呢？

Q：内存泄漏是什么？为什么会发生？常见哪些内存泄漏的例子？都是怎么解决的？

Q：内存泄漏和内存溢出的区别？

Q：什么情况会导致内存溢出？

#### 谷歌新动态

Q：是否了解和使用过谷歌推出的新技术？

Q：有了解刚发布的Androidx.0的特性吗？

Q：Kotlin对Java做了哪些优化？

## 3.2 Java

#### 基础

Q：面向对象编程的四大特性及其含义？

Q：String、StringBuffer和StringBuilder的区别？

Q：String a=""和String a=new String("")的的关系和异同？

Q：Object的equal()和==的区别？

Q：装箱、拆箱什么含义？

Q：int和Integer的区别？

Q：遇见过哪些运行时异常？异常处理机制知道哪些？

Q：什么是反射，有什么作用和应用？

Q：什么是内部类？有什么作用？静态内部类和非静态内部类的区别？

Q：final、finally、finalize()分别表示什么含义？

Q：重写和重载的区别？

Q：抽象类和接口的异同？

Q：为什么匿名内部类中使用局部变量要用final修饰？

Q：Object有哪些公用方法？

#### 集合

Q：Java集合框架中有哪些类？都有什么特点

Q：集合、数组、泛型的关系，并比较

Q：ArrayList和LinkList的区别？

Q：ArrayList和Vector的区别？

Q：HashSet和TreeSet的区别？

Q：HashMap和Hashtable的区别？

Q：HashMap在put、get元素的过程？体现了什么数据结构？

Q：如何解决Hash冲突？

Q：如何保证HashMap线程安全？什么原理？

Q：HashMap是有序的吗？如何实现有序？

Q：HashMap是如何扩容的？如何避免扩容？

Q：hashcode()的作用，与equal()有什么区别？

#### 并发

Q：开启一个线程的方法有哪些？销毁一个线程的方法呢？

Q：同步和非同步、阻塞和非阻塞的概念

Q：Thread的join()有什么作用？

Q：线程的有哪些状态？

Q：什么是线程安全？保障线程安全有哪些手段？

Q：ReentrantLock和synchronized的区别?

Q：synchronized和volatile的区别？

Q：synchronized同步代码块还有同步方法本质上锁住的是谁？为什么？

Q：sleep()和wait()的区别？

#### Java新动态

Q：是否了解Java1.x的特性吗？

Q：谈谈对面向过程编程、面向对象编程还有面向切面编程的理解

## 3.3 计算机网络

#### 基础

Q：五层协议的体系结构分别是什么？每一层都有哪些协议？

Q：为何有MAC地址还要IP地址？

#### TCP

Q：TCP和UDP的区别？

Q：拥塞控制和流量控制都是什么，两者的区别？

Q：谈谈TCP为什么要三次握手？为什么要四次挥手？

Q：播放视频用TCP还是UDP？为什么？

#### HTTP

Q：了解哪些响应状态码？

Q：get和post的区别？

Q：Http1.0、Http1.1、Http2.0的区别？

Q：HTTP和TCP的区别?

Q：HTTP和HTTPS的区别?

Q：HTTP和Socket的区别?

Q：在地址栏打入http://www.baidu.com会发生什么？

## 3.4 JVM

Q：JVM内存是如何划分的？

Q：谈谈垃圾回收机制？为什么引用计数器判定对象是否回收不可行？知道哪些垃圾
回收算法？

Q：Java中引用有几种类型？在Android中常用于什么情景？

Q：类加载的全过程是怎样的？什么是双亲委派模型？

Q：工作内存和主内存的关系？在Java内存模型有哪些可以保证并发过程的原子性、可见性和有序性的措施？

Q：JVM、Dalvik、ART的区别？

Q：Java中堆和栈的区别？

## 3.5 操作系统

Q：操作系统中进程和线程的区别？

Q：死锁的产生和避免?

## 3.6 数据结构&算法

Q：怎么理解数据结构？

Q：什么是斐波那契数列？

Q：迭代和递归的特点，并比较优缺点

Q：了解哪些查找算法，时间复杂度都是多少？

Q：了解哪些排序算法，并比较一下，以及适用场景

Q：快排的基本思路是什么？最差的时间复杂度是多少？如何优化？

Q：AVL树插入或删除一个节点的过程是怎样的？

Q：什么是红黑树？

Q：100盏灯问题

Q：老鼠和毒药问题，加个条件，必须要求第二天出结果

Q：海量数据问题

Q：（手写算法）二分查找

Q：（手写算法）反转链表

Q：（手写算法）用两个栈实现队列

Q：（手写算法）多线程轮流打印问题

Q：（手写算法）如何判断一个链有环/两条链交叉

Q：（手写算法）快速从一组无序数中找到第k大的数/前k个大的数

Q：（手写算法）最长（不）重复子串

## 3.7 设计模式

Q：谈谈MVC、MVP和MVVM，好在哪里，不好在哪里？

Q：如何理解生产者消费者模型？

Q：是否能从Android中举几个例子说说用到了什么设计模式？

Q：装饰模式和代理模式有哪些区别？

Q：实现单例模式有几种方法？懒汉式中双层锁的目的是什么？两次判空的目的又是什么？

Q：谈谈了解的设计模式原则？

## 3.8 数据库

Q：数据库中的事务了解吗？事务的四大特性？

Q：如何理解数据库的范式？

## 3.9 hr问题

Q：请简单的自我介绍一下

Q：谈谈项目经历，为什么会做，怎么做的，遇到的难点？

Q：谈谈实习经历，做了什么，收获有哪些？

Q：谈谈学习Android的经历，有哪些学习方法和技巧？

Q：是否会考研？/为何不保研？

Q：成绩怎么样？奖学金情况?

Q：学过哪些课程？那门课印象最深刻/最有意义/学的最好/最不喜欢？为什么？

Q：近x年的职业规划？

Q：为什么想来我们公司？/为何不转正留在xx?

Q：对公司/部门是否有了解？

Q：为何会选择做技术？/对女生做开发的看法？

Q：学习生活中遇到什么挫折，如何解决的？

Q：还投过那些公司，进展如何？如何xx和xx都给你发offer会如何选择？

Q：家是哪里的？是独生子女吗？从小的家庭环境如何？

Q：平常有哪些兴趣爱好？大学参加了哪些校园活动？

Q：有男/女朋友吗？未来有什么规划？

Q：评价一下自己的优缺点？/用x个词形容你自己。/别人都是怎样评价你的？

Q：觉得自己博客写的最好的文章是什么？为什么？

Q：觉得自己的优势是什么？

Q：如何看待加班？

Q：意向工作城市是哪？/是否会考虑在xx发展?

Q：对于薪酬有什么想法？

Q：有什么问题想要问我？

## 3.10 项目相关、实习相关技术问题（略）

Q：使用那些版本控制工具？Git和SVN的区别？

Q：了解Git工具吗？用过哪些命令？解决冲突时git merge和git rebase的区别？

点击此处见Android学习笔记清单：
https://www.jianshu.com/p/c44d7a106302