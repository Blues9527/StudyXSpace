## HandlerThread
- HandlerThread拥有自己的消息队列，它不会干扰或阻塞UI线程
- HandlerThread并不适用于网络IO操作，因为它只有一个线程，串行
- HandlerThread将loop转到子线程中处理，分担MainLooper的工作量，降低了主线程的压力，使主界面更流畅
- 可用HandlerThread实现子线程间的通信


> HandlerThread 继承自 Thread，本质是一个线程而不是一个handler。


> HandlerThread 执行方法 重写Thread的run() 方法

> 复写run()方法里主要实现的逻辑： 首先调用Looper.prepare()方法，初始化Looper，然后用synchronized来同步当前类，在synchronized代码块里获取从当前looper对象，然后调用notifyAl()l方法。接着设置线程的优先级，然后执行onLooperPrepared()空方法，再调用Looper.loop()方法开始工作。

```
@Override
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```


```
@Method

getLooper() Looper;

//如果handler为空，则new 一个handler
getThreadHandler() Handler;

//当消息队列里面没有消息时，判断Looper是否quit。
//首先通过getLooper()方法获取当前的looper对象，如果looper对象不为空，则调用looper.quit()方法去终止looper的操作，并返回true。，否则返回false。
quit() boolean;

//首先通过getLooper()方法获取当前的looper对象，如果looper对象不为空，则调用looper.quitSafely()方法去终止looper的操作，并返回true。，否则返回false。
quitSafely() boolean;

//返回Process.myTid()
getThreadId() int;
```
