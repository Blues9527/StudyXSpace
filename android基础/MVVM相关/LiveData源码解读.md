# LiveData

    LiveData是一个带泛型的抽象类。
    
实现类：
> MutableLiveData<T>
> MediatorLiveData

@Class LifecycleBoundObserver
> 继承于ObserverWrapper，并实现了GenericLifecycleObserver接口。

```
构造函数传进来一个LicfeCycleOwner

重写了shouldBeActive()，onStateChanged(),isAttachedTo(),detachObserver()等方法。

shouldBeActive()返回LifeCycle.isAtLeast(STARTED)

@params LifecycleOwner Lifecycle.Event
onStateChanged()
判断是否处于DESTROYED状态，是的话则移除Observer，否则执行父类的activeStateChanged()方法。

isAttachedTo()返回LifeCycleOwner是否一致；

detachObserver()移除Observer；
```

@Class ObserverWrapper
是一个私有的抽象类，LifecycleBoundObserver和AlwaysActiveObserver都是它的具体实现类。

```
抽象方法
shouldBeActive() boolean；

@params LifecycleOwner
isAttachedTo() boolean; //默认返回false

detachObserver() void;//空方法

@params boolean
activeStateChanged() void;
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