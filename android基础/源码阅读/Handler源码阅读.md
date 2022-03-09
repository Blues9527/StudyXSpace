
##### 构造函数
```
//不指定looper构造器
public Handler(Callback callback, boolean async) {
    //检测内存泄漏
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            //提示handler使用时应该用static修饰
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }
    //因为无传入looper，所以默认实例化一个looper
    //调用顺序Looper.myLooper() -> ThreadLocal.get() -> Looper.prepare() -> ThreadLocal.set(new Looper(boolean quitAllowed))
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        //如果没有调用过Looper.prepare()就会报出异常
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}


//有looper的构造函数
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

#### 核心方法

```
//handleMessage()内部是空实现，所以使用的时候必须覆写此方法去实现处理消息的逻辑
public void handleMessage(Message msg) {
}

//obtainMessage()是handler创建消息的方法，核心都是通过Message.obtain()去获取消息
//this指代handler，因为message必须指定handler，不然looper拿到消息的时候就不知道该给哪个handler处理消息了
public final Message obtainMessage(){
    return Message.obtain(this);
}

//异步发送消息
public final boolean post(Runnable r){
   return  sendMessageDelayed(getPostMessage(r), 0);
}

//消息插队
public final boolean postAtFrontOfQueue(Runnable r){
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}

//指定时间发送消息，也可以实现消息插队和消息延迟发送
public final boolean postAtTime(Runnable r, long uptimeMillis){
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}

//消息延迟发送
public final boolean postDelayed(Runnable r, long delayMillis){
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}

//同步的发送消息
public final boolean sendMessage(Message msg){
    return sendMessageDelayed(msg, 0);
}

//消息插队
public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}

//指定时间点发送消息
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

//延迟发送消息
public final boolean sendMessageDelayed(Message msg, long delayMillis){
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

```
//发送消息的核心
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    //调用MessageQueue的 enqueueMessage方法
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

```
//消息分发核心方法，也是looper中loop中的核心方法
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //最后都是交给handleMessage去处理逻辑，因为handler里的handleMessage是空实现，所以如果开发者要处理逻辑必须要重写此方法
        handleMessage(msg);
    }
}
```



##### post/sendMessage方法的调用流程

```

post(Runnable r)/sendMessage(Message msg)

-->sendMessageDelayed(Message msg, long delayMillis)

--> sendMessageAtTime(Message msg, long uptimeMillis)

--> enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)

--> MessageQueue的enqueueMessage(Message msg, long when)
```

#### Handler机制
> Handler 通过sendMessage/post方法发送Message给MessageQueue，通过enqueueMessage()方法压入MessageQueue，消息队列采用的是先进先出的数据结构，因为Handler初始化的时候会指定Looper，而Looper实例化的时候也会实例化对应的MessageQueue，所以Handler-MessageQueue-Looper就形成了一一对应的关系。在enqueueMessage方法中，有指定handler 'msg.target=this'，从而绑定了发送的Message与Handler的关系。Looper通过loop方法不断地从MessageQueue中将Message取出来，然后通过'msg.target.dispatchMessage()'方法将消息发送给指定Handler，其中msg.target就是指定的一个Handler对象。Handler通过handleMessage()方法去处理从looper中发送过来的消息。

#### Looper对应多个Handler的时候，Looper从MessageQueue中把消息取出来之后，如何知道要交给哪个Handler去处理呢？
> Handler发送消息的时候，最终会调用MessageQueue.enqueueMessage()方法，此时会对传入的Message进行赋值msg.target = this；这里的this指代就是当前Handler对象，所以Looper通过loop()方法从MessageQueue中把消息取出来，最终调用的是msg.target.disptachMessage(),通过msg.target就能知道要交由哪个Handler处理


#### 是否可以在子线程创建looper？如过可以那如何在子线程创建looper呢？
> 可以，调用Looper.prepare()方法，就会在当前线程创建一个looper，如果当前线程是子线程，就会在子线程创建一个looper，looper创建的时候也会实例化MessageQueue

#### 如何复用Message对象，为什么需要复用？
> 加入不复用Message对象，就会实例化Message对象，当消息处理完毕后就要回收这些Message对象，当用户频繁且快速操作的时候，可能会造成内存抖动。复用Message对象时，可以通过Handler的obtainMessage()方法去生成Message对象，也可以通过Message的obtain()去生成。

#### Handler产生内存泄漏的原因？如何防止Handler内存泄漏？
> 如果使用handler(非静态handler/匿名handler)去处理耗时操作的时候，当界面被销毁时handler还没有处理完耗时操作，就会产生内存泄漏。创建handler时使用static修饰，使handler为静态成员变量；创建静态内部类去继承Handler再使用；或者在使用完handler的时候，调用removeCallbacksAndMessages()移除所有回调和消息；使用WeakReference引用Handler。

