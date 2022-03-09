### 内存泄漏
#### 1.名词解释
内存泄漏：是指没有用的对象到GCRoots是可达的(即存在引用)，导致GC无法回收该对象，从而造成内存泄漏。

内存抖动：是指短时间内频繁创建对象，内存为了应对这种情况而进行频繁的GC。

#### 2.内存分析工具
1. Memory Monitor
2. Allocation Tracker
3. Heap Dump
4. MAT
5. [LeakCanary](https://github.com/square/leakcanary)


#### 3.内存泄漏产生的原因
1. 开发者编码造成的
2. 第三方框架造成的
3. Android系统或者第三方rom造成的

#### 4.内存泄漏的场景
1.非静态内部类的静态实例
```
原因：因为非静态内部类会持有外部类实例的引用，导致外部类无法被回收导致内存泄漏。

解决办法：用static修饰内部类，静态内部类不会持有外部类的引用
```

2.多线程相关的匿名内部类/非静态内部类
```
原因：匿名内部类也会持有外部类实例的引用，在多线程的情况下做耗时任务，会导致外部类无法被回收导致内存泄漏。

多线程相关的类有：AsyncTask、Thread、实现Runnable接口的类等

解决办法：自定义一个静态的类
```

3.Handler内存泄漏

```
原因：Handler的Message存储在MessageQueue中，有些Message并不能马上被处理，而且会在MessageQueue中存在很长的时间，导致Handler无法被回收，从而导致持有handler实例的Activity或者是Service不能被回收而导致内存泄漏

解决办法：一、使用静态的Handler内部类，内部使用弱引用去保存handler的实例；二、在Activity的destroy方法中移除MessageQueue中的消息，即 handler.removeCallbackAndMessage()，但是会导致消息无法完全被处理完。
```

4.context导致的内存泄漏

```
原因：对于非必须使用activity的context，当context被一个静态对象持有时，当屏幕发生旋转时上一个Activity被销毁，但是activity的实例依旧被持有着导致无法被回收

解决办法：使用application的context替代activity的context
```

5.静态view

```
原因：静态view可以避免每次加载的时候对view进行渲染，但是view会持有当前activity的实例引用导致activity无法被回收

解决方法：在activity的Destry中手动将view置空
```

6.Webview

```
原因：webview在不同版本的Android上存在很大的兼容问题，只要使用类webview内存就无法被释放掉。

解决办法：为webview单开一个线程，通过AIDL与应用住线程进行通信，根据业务需求在适当的时机进行销毁。
```

7.资源对象未关闭

```
原因：资源对象如Cursor、File等，往往都使用类缓冲，造成内存泄漏。在不实用的时候没有对资源进行关闭。

解决办法：在finally中进行关闭，置空
```

8.集合中对象没清理 

```
原因：将对象的引用添加至集合中，如果对象不需要类没有手动去清理引用从而导致集合越来越大，如果是static的情况会更加糟糕
```

9.Bitmap对象

```
原因：临时创建某个相对较大的bitmap对象，经过变换得到新的bitmap对象之后，没有即使回收旧的bitmap对象。

解决办法：避免静态变量持有较大的Bitmap对象或者其他大数据对象，如果真的持有类需要经快置空静态变量。
```

10.监听服务未关闭

```
原因：许多系统服务需要手动去注册与反注册，如果没有及时反注册，则肯能会导致内存泄漏

解决办法：在合适的时机对监听器进行反注册操作，如果是手动添加的listener，需要及时移除监听。
```




