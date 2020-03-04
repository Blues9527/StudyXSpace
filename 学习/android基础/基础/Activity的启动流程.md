[toc]

> android系统是基于linux内核的，而应用层则是基于Jvm的

# 主Activity的启动流程
### 1.ActivityThread
> ActivityThread中具有main()入口方法，Java程序入口就是main方法。

```
main方法里面做了什么？

//主线程初始化Looper
Looper.prepareMainLooper();
...
ActivityThread thread = new ActivityThread();
thread.attach(false, startSeq);
if (sMainThreadHandler == null) {
        //handler是ActivityThread的成员变量，一开始就实例化了，在这里获取到handler的实例并赋值给sMainThreadHandler
        sMainThreadHandler = thread.getHandler();
    }
...
//开启循环
Looper.loop();

```


### 2.Looper

```
//初始化主线程Looper
public static void prepareMainLooper() {
    //主线程Looper初始化，传入的false，即表示不可退出
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        //复制到此变量，调用getMainLooper()就返回主线程looper
        sMainLooper = myLooper();
    }
}

//普通线程Looper初始化
public static void prepare() {
    //普通线程looper可以退出
    prepare(true);
}

//looper退出的时候会判断quitAllowed
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //存放looper
    sThreadLocal.set(new Looper(quitAllowed));
}


public void quit() {
        //实际是MessageQueue判断的
        mQueue.quit(false);
    }
```

### MessageQueue

```
void quit(boolean safe) {
    //looper传过来的
    if (!mQuitAllowed) {
        //主线程Looper如果退出则会抛出异常
        throw new IllegalStateException("Main thread not allowed to quit.");
    }
    synchronized (this) {
        if (mQuitting) {
            return;
        }
        mQuitting = true;
        if (safe) {
            removeAllFutureMessagesLocked();
        } else {
            removeAllMessagesLocked();
        }
        // We can assume mPtr != 0 because mQuitting was previously false.
        nativeWake(mPtr);
    }
}
```

# 普通Activity的启动流程
> 由一个Activity正常启动另外一个Activity是通过startActivity()方法去启动的。


```
方法调用顺序

startActivity()

startActivityForResult()

Instrumentation.execStartActivity()

//IActivityManager本质是一个binder
(IActivityManager)ActivityManager.getService().startActivity()

//通过Binder去请求ActivityManagerService
ActivityManagerService.startActivity()

ActivityManagerService.startActivityAsUser()

ActivityStarter.startActivityMayWait()

//ActivityStarter里有3个startActivity()方法，是嵌套调用，也就是ActivityStarter内部调用里至少3个方法重载的startActivity()
ActivityStarter.startActivity()

ActivityStarter.startActivityUnchecked()

ActivityStackSupervisor.resumeFocusedStackTopActivityLocked()

ActivityStack.resumeTopActivityUncheckedLocked()

ActivityStack.resumeTopActivityInnerLocked()

//暂停当前Activity
ActivityStack.startPausingLocked()

//通过IPC请求到里ApplicationThread，ApplicationThread是Activity的一个内部类
//统一通过事务处理
ApplicationThread.scheduleTransaction()

//ActivityThread.scheduleTransaction()，但是没有这个方法，而是通过ClientTransactionHandler转发
//由此可知，ApplicationThread与ActivityThread通信是通过ClientTransactionHandler实现的
ActivityThread.handlePauseActivity()

//暂停当前activity
ActivityThread.performPauseActivity()

ActivityThread.performPauseActivityIfNeeded()

//又回到里Instrumentation
Instrumentation.callActivityOnPause()

//Instrumentation最终调用的是Activity的performPause()最终去暂停当前Activity
Activity.performPause()

//IPC to AMS
ActivityManagerService.activityPaused()

ActivityStack.activityPausedLocked()

ActivityStack.completePauseLocked()

ActivityStackSupervisor.resumeFocusedStackTopActivityLocked()

ActivityStack.resumeTopActivityUncheckedLocked()

ActivityStack.resumeTopActivityInnerLocked()

//上次执行的是ActivityStack.startPausingLocked()去暂停activity，这次是启动新的activity
ActivityStackSupervisor.startSpecificActivityLocked()

//此前会先判断进程是否为空，为空会执行ActivityManagerService.startProcessLocked()去启动一个线程，而当前情况下是已有进程里
//执行ActivityManagerService.startProcessLocked()应该是启动MainActivity时会进入
ActivityStackSupervisor.realStartActivityLocked()

//IPC 通过ClientLifecycleManager去请求ApplicationThread
//ClientLifecycleManager 会通过ClientTransaction去获取到IApplicationThread，然后进行IPC
ApplicationThread.scheduleTransaction()

//ActivityThread.scheduleTransaction()，实际上ActivityThread已经没有这个方法里，是通过ClientTransaction去调用 handleLaunchActivity()
ActivityThread.handleLaunchActivity()

ActivityThread.performLaunchActivity()

//----具体创建activity过程-------
//ActivityThread.performLaunchActivity()内调用
Instrumentation.newActivity()

//Instrumentation.newActivity()内调用
AppComponentFactory.instantiateActivity()

//AppComponentFactory.instantiateActivity()内调用
ClassLoader.loadclass().newInstance()

//ActivityThread.performLaunchActivity()内调用，activity创建完毕之后调用attach()方法，属性绑定
Activity.attach()


//--------onCreate()------------
//ActivityThread.performLaunchActivity()内调用
Instrumentation.callActivityOnCreate()

//Instrumentation.callActivityOnCreate()内调用
Activity.performCreate()

Activity.onCreate()

//onCreate执行完后执行onStart，通过ClientTransactionHandler通知执行的
ActivityThread.handleStartActivity()

//ActivityThread.handleStartActivity()内调用
Activity.performStart()

//--------onStart()--------
//Activity.performStart()内调用
Instrumentation.callActivityOnStart()

//Instrumentation.callActivityOnStart()内调用
Activity.onStart()

//-------onRestoreInstanceState------
//Activity.performStart()完之后执行，ActivityThread.handleStartActivity()内调用，不是必调的，是有条件的
Instrumentation.callActivityOnRestoreInstanceState()
->
Activity.performRestoreInstanceState()
->
Activity.onRestoreInstanceState()

//---------onPostCreate()------
//Instrumentation.callActivityOnRestoreInstanceState()完后执行，ActivityThread.handleStartActivity()内调用，不是必调的，是有条件的
Instrumentation.callActivityOnPostCreate()
->
Activity.onPostCreate()


//onStart完毕后，执行onResume,通过ClientTransactionHandler通知执行的
ActivityThread.handleResumeActivity()
->
ActivityThread.performResumeActivity()
->
Activity.performResume()
->
Instrumentation.callActivityOnResume()
->
Activity.onResume()

Activity.performResume()-> performRestart()
```

#### 附注一：IActivityManager源码
```Java
public static IActivityManager getService() {
    return IActivityManagerSingleton.get();
}

private static final Singleton<IActivityManager> IActivityManagerSingleton =
        new Singleton<IActivityManager>() {
            @Override
            protected IActivityManager create() {
                final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                //从这里可以看出
                final IActivityManager am = IActivityManager.Stub.asInterface(b);
                return am;
            }
        };
```

### 普通Activity启动流程 Q&A
1. Instrumentation的作用？
> 负责AppThread与Activity间交互的作用

### 普通Activity启动流程总结
暂停当前Activity
- step1:Activity请求Instrumentation
- step2:Instrumentation请求AMS
- step3:AMS请求ApplicationThread
- step4:ApplicationThread与ActivityThread交互
- step5:ActivityThread请求Instrumentation
- step6:Instrumentation请求Activity暂停Activity

启动目标Activity
- step1:AMS请求ApplicationThread去执行Transaction
- step2:ApplicationThread与ActivityThread交互
- step3:ActivityThread调用handleLaunchActivity()
- step4:ActivityThread与Instrumentation交互
- step5:Instrumentation与Activity交互，完成onCreate()

后续的onStart()、onResume()同样过程

> 通俗地将，从一个Activity去启动另外一个Activity需要两个步骤，可以从Activity的生命周期中可以体现，先是暂停当前Activity，然后再是启动另外一个Activity。

> Activity->Instrumentation->AMS->ApplicationThread->ActivityThread(handlePauseActivity)

> ActivitySupervisor(scheduleTransaction)->ApplicationThread->ActivityThread(launchActivity)

# 系统Activity启动与普通Activity启动的异同
> 系统Activity的启动主要可以分为3个步骤，launcher去请求AMS，然后AMS到ApplicationThread的调用，再到ActivityThread启动Activity

> 普通Activity的主要可以分为两个步骤，一是停止当前Activity，二是启动新的Activity。首先是Activity调用Instrumentation去请求AMS，然后AMS到ApplicationThread的调用，然后ActivityThread停止当前Activity，调用的handlePasueActivity()，然后ActivitySupervisor到ApplicationThread的调用，然后ActivityThread去启动Activity，调用的handleLaunchActivity()