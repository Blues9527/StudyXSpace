Observable这类as显示的总行数是 15491行，相比View来说（27753）小意思啦。粗略浏览了一下，虽然行数很多，但是，里面具有大量注释（良心源码）和大量的方法重载（同名方法）以及大量的自定义注解。


1.先看create方法
```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}

对应的onAssembly方法
@NonNull
public static <T> Observable<T> onAssembly(@NonNull Observable<T> source) {
    Function<? super Observable, ? extends Observable> f = onObservableAssembly;
    if (f != null) {
        return apply(f, source);
    }
    return source;
}

对应的Function类
public interface Function<T, R> {
    /**
    *   传入的泛型参数T，返回泛型R，会抛出Exception异常
    **/
    R apply(@NonNull T t) throws Exception;
}

用法，主要用于创建一个observable
Observable ob = Observable.create()
```

2.在看interval方法
```
interval方法使用起来就像一个定时任务那样，每隔一段时间执行一次操作

interval方法具有方法重载的特性

@SchedulerSupport(SchedulerSupport.COMPUTATION)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit) {
    return interval(initialDelay, period, unit, Schedulers.computation());
}

@NonNull
@SchedulerSupport(SchedulerSupport.CUSTOM)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler) {
    ObjectHelper.requireNonNull(unit, "unit is null");
    ObjectHelper.requireNonNull(scheduler, "scheduler is null");
    return RxJavaPlugins.onAssembly(new ObservableInterval(Math.max(0L, initialDelay), Math.max(0L, period), unit, scheduler));
}

@SchedulerSupport(SchedulerSupport.COMPUTATION)
public static Observable<Long> interval(long period, TimeUnit unit) {
    return interval(period, period, unit, Schedulers.computation());
}

@SchedulerSupport(SchedulerSupport.CUSTOM)
public static Observable<Long> interval(long period, TimeUnit unit, Scheduler scheduler) {
    return interval(period, period, unit, scheduler);
}
```

3.never方法

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
@SuppressWarnings("unchecked")
public static <T> Observable<T> never() {
    return RxJavaPlugins.onAssembly((Observable<T>) ObservableNever.INSTANCE);
}

```

4.observeOn方法

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.CUSTOM)
public final Observable<T> observeOn(Scheduler scheduler) {
    return observeOn(scheduler, false, bufferSize());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.CUSTOM)
public final Observable<T> observeOn(Scheduler scheduler, boolean delayError) {
    return observeOn(scheduler, delayError, bufferSize());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.CUSTOM)
public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
    ObjectHelper.requireNonNull(scheduler, "scheduler is null");
    ObjectHelper.verifyPositive(bufferSize, "bufferSize");
    return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
}
```

5.subscribe方法

```
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe() {
    return subscribe(Functions.emptyConsumer(), Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION, Functions.emptyConsumer());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext) {
    return subscribe(onNext, Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION, Functions.emptyConsumer());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {
    return subscribe(onNext, onError, Functions.EMPTY_ACTION, Functions.emptyConsumer());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
        Action onComplete) {
    return subscribe(onNext, onError, onComplete, Functions.emptyConsumer());
}

@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
        Action onComplete, Consumer<? super Disposable> onSubscribe) {
    ObjectHelper.requireNonNull(onNext, "onNext is null");
    ObjectHelper.requireNonNull(onError, "onError is null");
    ObjectHelper.requireNonNull(onComplete, "onComplete is null");
    ObjectHelper.requireNonNull(onSubscribe, "onSubscribe is null");
    LambdaObserver<T> ls = new LambdaObserver<T>(onNext, onError, onComplete, onSubscribe);
    subscribe(ls);
    return ls;
}

@SchedulerSupport(SchedulerSupport.NONE)
@Override
public final void subscribe(Observer<? super T> observer) {
    ObjectHelper.requireNonNull(observer, "observer is null");
    try {
        observer = RxJavaPlugins.onSubscribe(this, observer);
        ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null Observer. Please change the handler provided to RxJavaPlugins.setOnObservableSubscribe for invalid null returns. Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");
        subscribeActual(observer);
    } catch (NullPointerException e) { // NOPMD
        throw e;
    } catch (Throwable e) {
        Exceptions.throwIfFatal(e);
        // can't call onError because no way to know if a Disposable has been set or not
        // can't call onSubscribe because the call might have set a Subscription already
        RxJavaPlugins.onError(e);
        NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
        npe.initCause(e);
        throw npe;
    }
}
```

6.subscribeActual具体实现方法

```
protected abstract void subscribeActual(Observer<? super T> observer);
```

7.subscribeOn

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.CUSTOM)
public final Observable<T> subscribeOn(Scheduler scheduler) {
    ObjectHelper.requireNonNull(scheduler, "scheduler is null");
    return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
}
```

8.empty

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
@SuppressWarnings("unchecked")
public static <T> Observable<T> empty() {
    return RxJavaPlugins.onAssembly((Observable<T>) ObservableEmpty.INSTANCE);
}
```

9.error

```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> error(Callable<? extends Throwable> errorSupplier) {
    ObjectHelper.requireNonNull(errorSupplier, "errorSupplier is null");
    return RxJavaPlugins.onAssembly(new ObservableError<T>(errorSupplier));
}

@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> error(final Throwable exception) {
    ObjectHelper.requireNonNull(exception, "exception is null");
    return error(Functions.justCallable(exception));
}
```

10.defer

```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> defer(Callable<? extends ObservableSource<? extends T>> supplier) {
    ObjectHelper.requireNonNull(supplier, "supplier is null");
    return RxJavaPlugins.onAssembly(new ObservableDefer<T>(supplier));
}
```
11.fromArray

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
@NonNull
public static <T> Observable<T> fromArray(T... items) {
    ObjectHelper.requireNonNull(items, "items is null");
    if (items.length == 0) {
        return empty();
    }
    if (items.length == 1) {
        return just(items[0]);
    }
    return RxJavaPlugins.onAssembly(new ObservableFromArray<T>(items));
}
```









