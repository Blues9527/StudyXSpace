
# LifeCycle

    LifeCycle是一个抽象类，类里面有两个枚举类，分别是State和Event。
    
State(Enum)
> DESTROYED 标识对应Activity的onDestroy()方法调用后的状态

> INITIALIZED 标识对应Activity刚被创建，但是没有被接受之前的状态

> CREATED 标识对应Activity中的onCreate()方法被调用后，或者是onStop()方法被调用前的状态

> STARTED 标识对应Activity中的onStart()方法被调用后，或者是onPause()方法被调用前的状态

> RESUMED 标识对应Activity中的onResume()方法被调用后的状态

State里还有一个方法 isAtLeast() boolean,主要用于判断传进来的状态是否>=0;
 
Event(Enum)
> ON_CREATE 对应onCreate()

> ON_START 对应onStart()

> ON_RESUME 对应onResume()

> ON_PAUSE 对应onPause()

> ON_STOP 对应onStop()

> ON_DESTROY 对应onDestroy()

> ON_ANY 对应任意event

LifeCycle里面还有几个方法，如下：

```
添加观察者,抽象方法
@params @NonNull LifecycleObserver observer
addObserver() void;
*只能用于主线程
```

```
移除观察者，抽象方法
@params @NonNull LifecycleObserver observer
removeObserver() void;
*只能用于主线程
```

```
获取当前状态，抽象方法
getCurrentState() State;
*只能用于主线程
```

用法:

```
class TestObserver implements DefaultLifecycleObserver {
    @Override
    public void onCreate(LifecycleOwner owner) {
        // your code
    }
}

or

class TestObserver implements LifecycleObserver {
  @OnLifecycleEvent(ON_STOP)
  void onStopped() {}
}
```