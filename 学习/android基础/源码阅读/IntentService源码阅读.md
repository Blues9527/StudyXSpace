### IntentService
> IntentService继承自Service，是Service的子类，使用了abstract 进行修饰，所以再使用的时候需要使用子类去继承IntentService后才能使用。



#### 构造函数

```
public IntentService(String name) {
        super();
        mName = name;
    }
```

#### 属性

```
    //looper
    private volatile Looper mServiceLooper;
    //handler
    private volatile ServiceHandler mServiceHandler;
    //线程名称，实例化时传入
    private String mName;
    //是否重发
    private boolean mRedelivery;
```

#### method

==setIntentRedelivery==
```
//用于设置是否需要重发
public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }
 ```
 
 ==onCreate==
 ```

//覆写父类Service的onCreate()方法，主要实例化一个HandlerThread并启动，还获取Looper对象并实例化ServiceHandler
@Override
public void onCreate() {
    super.onCreate();
    //实例化一个线程，HandlerThread的本质是一个Thread
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    //开启线程
    thread.start();

    //在HandlerThread的run方法中已经初始化过Looper了，所以可以通过getLooper()方法去获取当前的Looper
    mServiceLooper = thread.getLooper();
    //实例化一个handler，传入一个looper
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

```
==onStart==
```
//覆写父类的onStart()方法，主要实例化一个Message，然后调用sendMessage将Message发送出去
@Override
public void onStart(@Nullable Intent intent, int startId) {
    //实例化一个Message
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    //发送消息
    mServiceHandler.sendMessage(msg);
}
```

==onStartCommand==
```
//覆写父类的onStartCommand()方法
@Override
public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
    //内部调用onStart()方法
    onStart(intent, startId);
    return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
}
```

==onDestroy==
```
//覆写父类的onDestroy()方法
@Override
public void onDestroy() {
    //停止looper
    mServiceLooper.quit();
}
```

==onBind==
```
//覆写父类的onBind()方法
@Override
@Nullable
public IBinder onBind(Intent intent) {
    return null;
}
```

==onHandleIntent==
```
// 抽象方法，子类需要覆写此方法去实现耗时操作的逻辑
@WorkerThread
protected abstract void onHandleIntent(@Nullable Intent intent);
```

### ServiceHandler内部类
> ServiceHandler继承自Handler类，是一个Handler

#### 构造函数

```
public ServiceHandler(Looper looper) {
    super(looper);
}
```

#### method

```
//覆写父类的handleMessage方法
@Override
public void handleMessage(Message msg) {
    //内部是调用ServiceIntent的onHandleIntent()方法
    onHandleIntent((Intent)msg.obj);
    //调用Service的stopSelf方法，即执行完耗时操作后停止
    stopSelf(msg.arg1);
}
```

### 技术总结
1. IntentService默认不执行绑定操作，如果需要绑定操作，需要覆写onBind()方法去实现绑定逻辑
2. IntentService中能实现耗时操作是因为在onCreate中开启了一个HandlerThreead线程，而HandlerThead中在run方法创建一个Looper，所以可以在子线程中达到耗时操作的目的。
3. ServiceHandler的本质就是Handler，覆写了父类的handleMessage()方法调用了IntentService的抽象方法onHandleIntent()

