[toc]

# 重读Glide源码

## Glide的简单使用

```
Glide.with(context)
    //加载的目标url
    .load(string)
    //占位图
    .placeholder(int)
    //加载错误时显示的图片
    .error(int)
    //压缩
    .thumbnail(float)
    //存储策略
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    //跳过内存缓存
    .skipMemoryCache(boolean)
    //转场动画
    .transition(TransitionOptions)
    //裁剪变换支持Rotate、FitCenter、CircleCrop、CenterInside、CenterCrop、RoundedCorners
    .transform(new RoundedCorners(float))
    //加载监听
    .listener(listener)
    //加载到目标view
    .into(target);
```

## Glide的调用链

```java
//调用链
Glide.with()

RequeManager.load()

BaseRequestOptions.override()
BaseRequestOptions.centerCrop()
BaseRequestOptions.circleCrop()
RequestBuilder.transition()

RequestBuilder.into()


RequestBuilder.into(){
    //...
    requestManager.track(target, request);
}

RequestManager.track(){
    //...
    requestTracker.runRequest(request);
}

RequestTracker.runRequest(){
    //...
    request.begin();
}

SingleRequest.begin(){
    //...
     onSizeReady(...);
}

SingleRequest.onSizeReady(){
    //...
    engine.load()
}

Engine.load(){
    //...
    loadFromMemory()
    
    //...
    waitForExistingOrStartNewJob()
}

Engine.waitForExistingOrStartNewJob(...){
    //...
    engineJobFactory.build(...)
    
    //...
    decodeJobFactory.build(...)
    
    //...
    engineJob.addCallback(...)
    engineJob.start(...)
}

//执行请求
EngineJob.start(decodeJob){
    //...
    executor.execute(decodeJob)
}

DecodeJob.run(){
    //...
    runWrapped()
}

DecodeJob.runWrapped(){
    //...
    runGenerators()
}

DecodeJob.runGenerators(){
    //...
    reschedule()
}

DecodeJob.reschedule(){
    getActiveSourceExecutor().execute(job)
}

DecodeJob.runWrapped(){
    //...
    runGenerators()
}

DecodeJob.runGenerators(){
    //...
    currentGenerator.startNext()
}

SourceGenerator.startNext(){
    //...
    startNextLoad(loadData)
}

SourceGenerator.startNextLoad(){
    //...
    loadData.fetcher.loadData(...,{
        onDataReady(){
            onDataReadyInternal(toStart, data)
        }
        onLoadFailed(){
            onLoadFailedInternal(toStart, e)
        }
    })
}

MultiModelLoader.loadData(){
    //...
    fetchers.get(currentIndex).loadData(priority, this)
}

HttpUrlFetecher.loadData(){
    //...
    InputStream result = loadDataWithRedirects(glideUrl.toURL(), 0, null, glideUrl.getHeaders())
    callback.onDataReady(result)
}

MultiModelLoader.onDataReady(){
    //...
    callback.onDataReady(data)
}

SourceGenerator.onDataReadyInternal(){
    //...
    cb.reschedule();
}

DecodeJob.reschedule(){
    //...
    callback.reschedule(this)
}

DecodeJob.run(){
    //...
    runWrapped()
}

DecodeJob.runGenerators(){
    //...
    currentGenerator.startNext()
}

SourceGenerator.startNext(){
    //...
    sourceCacheGenerator.startNext()
}

DataCacheGenerator.startNext(){
    //...
    loadData.fetcher.loadData(helper.getPriority(), this)
}

ByteBufferFileLoader.loadData(){
    //...
    result = ByteBufferUtil.fromFile(file)
    callback.onDataReady(result)
}

DataCacheGenerator.onDataReady(){
    cb.onDataFetcherReady(...)
}

SourceGenerator.onDataFetcherReady(){
    cb.onDataFetcherReady(...)
}

DecodeJob.onDataFetcherReady(){
    //...
    decodeFromRetrievedData()
}

DecodeJob.decodeFromRetrievedData(){
    //...
    resource = decodeFromData(...)
    //...
    notifyEncodeAndRelease(...)
}

DecodeJob.notifyEncodeAndRelease(){
    //...
    notifyComplete(...)
}

DecodeJob.notifyComplete(){
    //...
    callback.onResourceReady(...)
}

EngineJob.onResourceReady(){
    //...
    notifyCallbacksOfResult()
}

EngineJob.notifyCallbacksOfResult(){
    //...
    incrementPendingCallbacks(copy.size() + 1)
    //...
    engineJobListener.onEngineJobComplete(...)
}

Engine.onEngineJobComplete(){
    //...
    activeResources.activate(key, resource)
    jobs.removeIfCurrent(key, engineJob)
}

ActiveResources.activate(){
    //更新缓存
    ResourceWeakReference removed = activeEngineResources.put(key, toPut);
    if (removed != null) {
      removed.reset();
    }
}


//将图片设置到imageview
EngineJob.addCallback(cb, callbackExecutor){
    if (hasResource) {
      incrementPendingCallbacks(1);
      
      //执行的CallResourceReady是一个runnable
      callbackExecutor.execute(new CallResourceReady(cb));
    }
}

//看一下里面的run方法
@Override
public void run() {
  synchronized (cb.getLock()) {
    synchronized (EngineJob.this) {
      if (cbs.contains(cb)) {
        engineResource.acquire();
        
        //继续回调
        callCallbackOnResourceReady(cb);
        removeCallback(cb);
      }
      decrementPendingCallbacks();
    }
  }
}

//继续往下看
@Synthetic
@GuardedBy("this")
void callCallbackOnResourceReady(ResourceCallback cb) {
    try {
    //还是回调，继续看
      cb.onResourceReady(engineResource, dataSource, isLoadedFromAlternateCacheKey);
    } catch (Throwable t) {
      throw new CallbackException(t);
    }
}

//具体实现只有SingleRequest
SingleReques.onResourceReady(...){
    //...
    onResourceReady(
            (Resource<R>) resource, (R) received, dataSource, isLoadedFromAlternateCacheKey)
}

//看另一个onResourceReady()
SingleReques.onResourceReady(...){

    //...
    if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation = animationFactory.build(dataSource, isFirstResource);
        //看一下具体实现
        target.onResourceReady(result, animation);
      }
}

//找到具体实现ImageViewTarget
@Override
public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
    if (transition == null || !transition.transition(resource, this)) {
    //继续看这个
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
}

private void setResourceInternal(@Nullable Z resource) {
//这是抽象方法，再看一下具体实现
    setResource(resource);
    maybeUpdateAnimatable(resource);
}

//BitmapImageViewTarget.setResource()
@Override
protected void setResource(Bitmap resource) {
    view.setImageBitmap(resource);
}

//DrawableImageViewTarget.setResource()
@Override
protected void setResource(@Nullable Drawable resource) {
    view.setImageDrawable(resource);
}

//ThumbnailImageViewTarget.setResource()
@Override
protected void setResource(@Nullable T resource) {
    ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
    Drawable result = getDrawable(resource);
    if (layoutParams != null && layoutParams.width > 0 && layoutParams.height > 0) {
      result = new FixedSizeDrawable(result, layoutParams.width, layoutParams.height);
    }
    
    view.setImageDrawable(result);
}
```


## Glide是如何切线程的？

### 怎么切出去的？
> 上面我们分析到了SingleReuqest的begin()方法，接着看

- GlideBuilder.build()
```java
@NonNull
Glide build(@NonNull Context context) {
    if (sourceExecutor == null) {
      sourceExecutor = GlideExecutor.newSourceExecutor();
    }

    if (diskCacheExecutor == null) {
      diskCacheExecutor = GlideExecutor.newDiskCacheExecutor();
    }
    
    //....省略代码....
    
    if (engine == null) {
      engine =
          new Engine(
              memoryCache,
              diskCacheFactory,
              diskCacheExecutor,
              sourceExecutor,
              GlideExecutor.newUnlimitedSourceExecutor(),
              animationExecutor,
              isActiveResourceRetentionAllowed);
    }
    
    //....省略代码....
    
    return new Glide(
        context,
        engine,
        memoryCache,
        bitmapPool,
        arrayPool,
        requestManagerRetriever,
        connectivityMonitorFactory,
        logLevel,
        defaultRequestOptionsFactory,
        defaultTransitionOptions,
        defaultRequestListeners,
        experiments);
    
}
```

- Engine()构造方法
> 创建EngineJobFactory

```
//....省略前面代码....

if (engineJobFactory == null) {
  engineJobFactory =
      new EngineJobFactory(
          diskCacheExecutor,
          sourceExecutor,
          sourceUnlimitedExecutor,
          animationExecutor,
          /*engineJobListener=*/ this,
          /*resourceListener=*/ this);
}

//....省略后面代码....
```

- EngineJobFactory
> 创建EngineJob

```
@Synthetic
final Pools.Pool<EngineJob<?>> pool =
    FactoryPools.threadSafe(
        JOB_POOL_SIZE,
        new FactoryPools.Factory<EngineJob<?>>() {
          @Override
          public EngineJob<?> create() {
            return new EngineJob<>(
                diskCacheExecutor,
                sourceExecutor,
                sourceUnlimitedExecutor,
                animationExecutor,
                engineJobListener,
                resourceListener,
                pool);
          }
        });
```

- EngineJob.start()

```java
public synchronized void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    GlideExecutor executor =
        decodeJob.willDecodeFromCache() ? diskCacheExecutor : getActiveSourceExecutor();
    //默认走的是diskCacheExecutor
    executor.execute(decodeJob);
}
```



### 怎么切回来的？

- RequestBuilder.into()
```java
//回调主线程，在into方法里
public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    return into(
        glideContext.buildImageViewTarget(view, transcodeClass),
        null,
        requestOptions,
        Executors.mainThreadExecutor());//这里设置了回调要执行的线程
}
//调用另一个into
private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
      
      //...
      Request request = buildRequest(target, targetListener, options, callbackExecutor);
      //...
      requestManager.clear(target);
      target.setRequest(request);
      requestManager.track(target, request);
      //...

```

- ReqeustManager.track()
```
synchronized void track(@NonNull Target<?> target, @NonNull Request request) {
    targetTracker.track(target);
    requestTracker.runRequest(request);
}
```

- RequestTracker.runRequest()

```java
public void runRequest(@NonNull Request request) {
    requests.add(request);
    if (!isPaused) {
      request.begin();//调用SingleRequest的begin()
    } else {
      request.clear();
      //...
      pendingRequests.add(request);
    }
}
```


- SingleRequest

```

//begin()
if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
    onSizeReady(overrideWidth, overrideHeight);
} else {
    target.getSize(this);
}

//onSizeReady()
loadStatus = engine.load(...,callbackExecutor);

//...


```


- Engine

```java
//load()
if (memoryResource == null) {
        return waitForExistingOrStartNewJob(...
            callbackExecutor,...);
      }

//waitForExistingOrStartNewJob()

engineJob.addCallback(cb, callbackExecutor);
engineJob.start(decodeJob);

//...
```


- EngineJob.addCallback()
```
synchronized void addCallback(final ResourceCallback cb, Executor callbackExecutor) {
    stateVerifier.throwIfRecycled();
    cbs.add(cb, callbackExecutor);
    if (hasResource) {
      incrementPendingCallbacks(1);
      //这里就是通过主线程执行
      callbackExecutor.execute(new CallResourceReady(cb));
    } else if (hasLoadFailed) {
      incrementPendingCallbacks(1);
      callbackExecutor.execute(new CallLoadFailed(cb));
    } else {
      Preconditions.checkArgument(!isCancelled, "Cannot add callbacks to a cancelled EngineJob");
    }
}
```



## Glide是如何感应生命周期的？
### LifecycleListener
> 用于监听Activity/Fragment的生命周期相关事件

```Java

void onStart();//对应Fragment.onStart()和Activity.onStart()

void onStop();//对应Fragment.onStop()和android.app.Activity.onStop()

void onDestroy();//对应Fragment.onDestroy()和Activity.onDestroy()
```

### Target
> xxTarget的基类接口，继承了LifecycleListener接口

#### ImageViewTarget
> 有LifecycleListener具体的实现，主要用于实现带动画效果的图片加载或加载动图。

### TargetTracker
> 也实现了LifecycleListener接口，为RequestManager保存当前处于活跃状态的Target集合，并转发生命周期。即此处实现的onStart/onStop/onDestroy会间接调用Target的onStart/onStop/onDestroy

### Lifecycle
> 用于监听Activity/Fragment的生命周期

```Java
void addListener(@NonNull LifecycleListener listener);//添加监听

void removeListener(@NonNull LifecycleListener listener);//移除监听
```
### ActivityFragmentLifecycle
> 实现了Lifecycle接口的管理类，内部使用Set来管理和存储LifecycleListener，统一处理onStart/onStop/onDestroy

### SupportRequestManagerFragment/RequestManagerFragment(已被标注为废弃)
 
>两者都是继承自Fragment的类，其中Fragment是一个view-less(我的理解就是载体Fragment，利用Fragment的生命周期函数onStart/onStop/onDestroy来管理Glide的request)
    
- onStart就是调用ActivityFragmentLifecycle的onStart，操作共2步，分别是标识isStarted = true，然后提取LifecycleListener set快照遍历然后执行onStart()
- onStop就是调用ActivityFragmentLifecycle的onStop，操作共2步，分别是标识isStarted = false，然后提取LifecycleListener set快照遍历然后执行onStop()
- onDestroy就是调用ActivityFragmentLifecycle的onDestroy，操作共2步，分别是标识isDestroyed = true，然后提取LifecycleListener set快照遍历然后执行onDestroy()。然后执行unregisterFragmentWithRoot()进行反注册，移除正在请求的Fragment载体

### RequestManager
> 实现了LifecycleListener接口，并且具体化了onStart/onStop/onDestroy的操作

- onStart
```Java
//同步操作
@Override
public synchronized void onStart() {
    //恢复请求
    resumeRequests();
    targetTracker.onStart();
}
```
- onStop
```Java
@Override
public synchronized void onStop() {
    //暂停请求
    pauseRequests();
    targetTracker.onStop();
}
```

- onDestroy
```Java
@Override
public synchronized void onDestroy() {
    
    targetTracker.onDestroy();
    for (Target<?> target : targetTracker.getAll()) {
      clear(target);
    }
    targetTracker.clear();
    //清空请求
    requestTracker.clearRequests();
    //移除生命周期监听
    lifecycle.removeListener(this);
    //移除网络监听
    lifecycle.removeListener(connectivityMonitor);
    Util.removeCallbacksOnUiThread(addSelfToLifecycle);
    //反注册请求管理器
    glide.unregisterRequestManager(this);
}
```

### RequestManagerRetriever
> 管理SupportRequestManagerFragment的一个类，通过HashMap来存储SupportRequestManagerFragment

```Java
  @NonNull
  private SupportRequestManagerFragment getSupportRequestManagerFragment(
      @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
    
    //通过tag去找fragment
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    //找不到
    if (current == null) {
      //从hashmap去找
      current = pendingSupportRequestManagerFragments.get(fm);
      //还找不到，就创建一个
      if (current == null) {
        current = new SupportRequestManagerFragment();
        //绑定fragment
        current.setParentFragmentHint(parentHint);
        //放入hashmap
        pendingSupportRequestManagerFragments.put(fm, current);
        //开始fragment事务
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        //通过handler发送消息
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }
  
  
  //看一下handler的handle message的实现
  @Override
  public boolean handleMessage(Message message) {
    boolean handled = true;
    Object removed = null;
    Object key = null;
    switch (message.what) {
      case ID_REMOVE_FRAGMENT_MANAGER:
        android.app.FragmentManager fm = (android.app.FragmentManager) message.obj;
        key = fm;
        removed = pendingRequestManagerFragments.remove(fm);
        break;
      case ID_REMOVE_SUPPORT_FRAGMENT_MANAGER:
        //这里就是处理上面发送出去的消息
        FragmentManager supportFm = (FragmentManager) message.obj;
        key = supportFm;
        //从hashmap中移除掉对应的fragment
        removed = pendingSupportRequestManagerFragments.remove(supportFm);
        break;
      default:
        handled = false;
        break;
    }
    return handled;
  }
```

### 小结
1.通过ConnectivityManager来监听网络的变化并通过广播来通知网络的变化
2.通过view-less的Fragment(具体实现是SupportRequestManagerFragment)来管理请求的生命周期

## Glide的缓存机制是怎样子的？
> Glide的缓存机制是分为内存缓存(memory cache)和磁盘缓存(disk cache)，其中内存缓存也分成使用中的缓存(弱引用缓存)和使用完的缓存(Lru缓存)

### 从缓存中取

#### Engine.load()
```java
public <R> LoadStatus load(...){
  //...
  synchronized (this) {
    //1.先从内存中取
      memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);

        //2.内存取不到，就从磁盘取或者网络下载
      if (memoryResource == null) {
        return waitForExistingOrStartNewJob(...);
      }
    }
  
}
```

#### Engine.loadFromMemory()
```java
@Nullable
private EngineResource<?> loadFromMemory(
  EngineKey key, boolean isMemoryCacheable, long startTime) {
    //关闭了内存缓存，返回null
    if (!isMemoryCacheable) {
      return null;
    }
    
    //1.从active resources中取
    EngineResource<?> active = loadFromActiveResources(key);
    if (active != null) {
      //...
      return active;
    }
    
    //2.从cache中取
    EngineResource<?> cached = loadFromCache(key);
    if (cached != null) {
      //...
      return cached;
    }
    
    return null;
}
```

#### Engine.loadFromActiveResources()
> 这里就是所谓的弱引用缓存

```java
@Nullable
private EngineResource<?> loadFromActiveResources(Key key) {
    EngineResource<?> active = activeResources.get(key);
    if (active != null) {
      active.acquire();
    }
    
    return active;
}
```

##### ActiveResources

```java
final class ActiveResources {
    
    final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();

    //放入弱引用缓存
    synchronized void activate(Key key, EngineResource<?> resource) {
        ResourceWeakReference toPut =
            new ResourceWeakReference(
                key, resource, resourceReferenceQueue, isActiveResourceRetentionAllowed);
    
        ResourceWeakReference removed = activeEngineResources.put(key, toPut);
        if (removed != null) {
          removed.reset();
        }
  }
  
    //移除弱引用缓存
    synchronized void deactivate(Key key) {
        ResourceWeakReference removed = activeEngineResources.remove(key);
        if (removed != null) {
          removed.reset();
        }
  }
  
    //从弱引用缓存中取
    @Nullable
    synchronized EngineResource<?> get(Key key) {
        ResourceWeakReference activeRef = activeEngineResources.get(key);
        if (activeRef == null) {
          return null;
        }
    
        EngineResource<?> active = activeRef.get();
        if (active == null) {
          cleanupActiveReference(activeRef);
        }
        return active;
  }
}

```

#### Engine.loadFromCache()
```java
private EngineResource<?> loadFromCache(Key key) {
    //从缓存中取
    EngineResource<?> cached = getEngineResourceFromCache(key);
    if (cached != null) {
        //取到了，放入弱引用缓存，++acquire
      cached.acquire();
      activeResources.activate(key, cached);
    }
    return cached;
}
```

##### Engine.getEngineResourceFromCache()
> 这里就是从Lru中取

```java
private EngineResource<?> getEngineResourceFromCache(Key key) {
    //这里的cache就是LruResourceCache
    Resource<?> cached = cache.remove(key);
    
    final EngineResource<?> result;
    if (cached == null) {
      result = null;
    } else if (cached instanceof EngineResource) {
      // Save an object allocation if we've cached an EngineResource (the typical case).
      result = (EngineResource<?>) cached;
    } else {
      result =
          new EngineResource<>(
              cached, /*isMemoryCacheable=*/ true, /*isRecyclable=*/ true, key, /*listener=*/ this);
    }
    return result;
}

```
#### Engine.waitForExistingOrStartNewJob()

```java
private <R> LoadStatus waitForExistingOrStartNewJob(...){

    EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
    if (current != null) {
      current.addCallback(cb, callbackExecutor);
      //...
      return new LoadStatus(cb, current);
    }
    
    //创建EngineJob
    EngineJob<R> engineJob = engineJobFactory.build(...);
    
    //创建DecodeJob，DecodeJob是一个runnable
    DecodeJob<R> decodeJob = decodeJobFactory.build(...);
    
    jobs.put(key, engineJob);
    
    engineJob.addCallback(cb, callbackExecutor);
    
    //进行网络请求
    engineJob.start(decodeJob);
    
    return new LoadStatus(cb, engineJob);
}

```

#### DecodeJob.run()
> DecodeJob本质就是一个runnable，在start()方法里放入executor执行


```
@Override
public void run() {

    //...   
      if (isCancelled) {
        notifyFailed();
        return;
      }
      
      runWrapped();
    //...
        
}
```



### 放入缓存
> 存放顺序应该是 先放入disk cache(如果允许)，再放入内存的弱引用缓存，当使用完了再放入Lru缓存

#### 缓存到disk cache
- SourceGenerator.startNext()
```java

@Override
public boolean startNext() {

    //...
    if (dataToCache != null) {
      Object data = dataToCache;
      dataToCache = null;
      //缓存数据
      cacheData(data);
    }
}
```

- SourceGenerator.cacheData()

```java
private void cacheData(Object dataToCache) {
    //...
      Encoder<Object> encoder = helper.getSourceEncoder(dataToCache);
      DataCacheWriter<Object> writer =
          new DataCacheWriter<>(encoder, dataToCache, helper.getOptions());
      originalKey = new DataCacheKey(loadData.sourceKey, helper.getSignature());
      
      //将缓存放入DiskLruCache
      helper.getDiskCache().put(originalKey, writer);
}
```

#### 缓存到内存(弱引用)

- Engine.onEngineJobComplete()
```java
@Override
public synchronized void onEngineJobComplete(
  EngineJob<?> engineJob, Key key, EngineResource<?> resource) {
    // A null resource indicates that the load failed, usually due to an exception.
    if (resource != null && resource.isMemoryCacheable()) {
      activeResources.activate(key, resource);
    }
    
    jobs.removeIfCurrent(key, engineJob);
}

//ActiveResources.activate()
synchronized void activate(Key key, EngineResource<?> resource) {

    //使用弱引用包裹一层
    ResourceWeakReference toPut =
        new ResourceWeakReference(
            key, resource, resourceReferenceQueue, isActiveResourceRetentionAllowed);
    
    //放入hashmap
    ResourceWeakReference removed = activeEngineResources.put(key, toPut);
    if (removed != null) {
      removed.reset();
    }
}
```

#### 缓存到内存(LRU)

- EngineResources.release()

```java
void release() {
    boolean release = false;
    synchronized (this) {
      if (acquired <= 0) {
        throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
      }
      if (--acquired == 0) {
        release = true;
      }
    }
    if (release) {
    //调用Engine.onResourceReleased()
      listener.onResourceReleased(key, this);
    }
}

```


- Engine.onResourceReleased()
```java
@Override
public void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
    //从弱引用缓存中移除
    activeResources.deactivate(cacheKey);
    if (resource.isMemoryCacheable()) {
    //如果内存缓存可用就放入LruCache，否则调用MemoryCacheAdapter的put方法(其内部实现是recycle())进行回收操作
      cache.put(cacheKey, resource);
    } else {
      resourceRecycler.recycle(resource, /*forceNextFrame=*/ false);
    }
}  

synchronized void deactivate(Key key) {
    ResourceWeakReference removed = activeEngineResources.remove(key);
    if (removed != null) {
    //置空并清除引用
      removed.reset();
    }
}
```



## Glide是怎么判断图片资源是在使用中还是已经使用完了，怎么做回收的？
> Glide 是通过EngineResource中的acquired变量来做使用记录的。

#### EngineResource.acquire()
```java
synchronized void acquire() {
    if (isRecycled) {
      throw new IllegalStateException("Cannot acquire a recycled resource");
    }
    ++acquired;
}
```

#### EngineResource.release()

```java
void release() {
    boolean release = false;
    synchronized (this) {
      if (acquired <= 0) {
        throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
      }
      if (--acquired == 0) {
        release = true;
      }
    }
    if (release) {
    //如果内存缓存可用，就会放入lrucache，否则就recycle
      listener.onResourceReleased(key, this);
    }
}
```

#### EngineResource.recycle()

```
@Override
public synchronized void recycle() {
    if (acquired > 0) {
      throw new IllegalStateException("Cannot recycle a resource while it is still acquired");
    }
    if (isRecycled) {
      throw new IllegalStateException("Cannot recycle a resource that has already been recycled");
    }
    isRecycled = true;
    if (isRecyclable) {
      resource.recycle();
    }
}
```

> 看一下具体的实现，以BitmapResource为例子

```java
//BitmapResource.recycle()
@Override
public void recycle() {
    //将bitmap放入bitmap pool以便复用
    bitmapPool.put(bitmap);
}
```

> bitmapPool是一个接口，具体实例化的是LruBitmapPool

```java
//GlideBuilder.build()
@NonNull
Glide build(@NonNull Context context) {
    
    //...
    
    if (bitmapPool == null) {
      int size = memorySizeCalculator.getBitmapPoolSize();
      if (size > 0) {
      //默认走的是这里
        bitmapPool = new LruBitmapPool(size);
      } else {
        bitmapPool = new BitmapPoolAdapter();
      }
    }
    
    //...
}
```

