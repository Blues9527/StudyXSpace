# LiveData

    LiveData是一个带泛型的抽象类。
    
@Class LifecycleBoundObserver
> 继承于ObserverWrapper，并实现了GenericLifecycleObserver接口。

```
构造函数传进来一个LicfeCycleOwner
LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<T> observer) {
            super(observer);
            mOwner = owner;
        }

重写了shouldBeActive()，onStateChanged(),isAttachedTo(),detachObserver()等方法。

@Override
        boolean shouldBeActive() {
		//调用LifeCycle.isAtLeast(STARTED),判断是否应该被激活
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

@Override
public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
	//判断是否处于DESTROYED状态
    if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
		//移除Observer
        removeObserver(mObserver);
        return;
    }
	//执行父类的activeStateChanged()方法
    activeStateChanged(shouldBeActive());
}


@Override
boolean isAttachedTo(LifecycleOwner owner) {
	//返回LifeCycleOwner是否一致；
    return mOwner == owner;
}


@Override
void detachObserver() {
	//移除Observer；
    mOwner.getLifecycle().removeObserver(this);
}
```

@Class ObserverWrapper
是一个私有的抽象类(包装类)，LifecycleBoundObserver和AlwaysActiveObserver都是它的具体实现类。

```
shouldBeActive() boolean；//抽象方法

boolean isAttachedTo(LifecycleOwner owner) {
			//默认返回false
            return false;
        } 

detachObserver() void;//空方法

void activeStateChanged(boolean newActive) {
    if (newActive == mActive) {
		//已处于激活状态，return
        return;
    }
	//赋值是否要激活
    mActive = newActive;
	//判断当前是否处于闲置状态，活跃数量为0则处于闲置状态
    boolean wasInactive = LiveData.this.mActiveCount == 0;
	//根据是否需要激活进行+1或者-1
    LiveData.this.mActiveCount += mActive ? 1 : -1;
	//同时满足，目前处于闲置状态且需要激活才为true
    if (wasInactive && mActive) {
		//执行激活操作
        onActive();
    }
    if (LiveData.this.mActiveCount == 0 && !mActive) {
        onInactive();
    }
    if (mActive) {
        dispatchingValue(this);
    }
```

@Class AlwaysActiveObserver
是一个私有类，继承于ObserverWrapper。

```
重写父类的shouldBeActive()并返回true；
```

LiveData的一些方法
```
@params ObserverWrapper
considerNotify() void;//private

@params ObserverWrapper
dispatchingValue()//private

@params LifecycleOwner Observer<T>
observe() void;//public 只能在mainThread调用。
首先判断当前状态是否是DESTROYED状态，是的话则return；否则就实例化一个LifecycleBoundObserver，然后获取一个ObserverWrapper的实力，判断两个实力是否为空，不为空的话则执行addObserver()方法去添加观察。

@params Observer<T>
observeForever() void;//public 只能在mainThread调用。

@params Observer<T>
removeObserver() void;//public  只能在mainThread调用。

@params LifecycleOwner
removeObservers() void;//public 只能在mainThread调用。
遍历所有的observers，判断它们是否是isAttachTo(LifecycleOwner),是的话则removeObserver();

@params T
postValue() void;//protected 在子线程调用

@params T
setValue() void;//protected 
内部调用的是dispatchingValue(null)方法

getValue() T;//public

getVersion() int;

onActive() void;//protected 空方法

onInactive() void;//protected 空方法

hasObservers() boolean;//public return观察者数量是否大于0；

hasActiveObservers() boolean;//public
```