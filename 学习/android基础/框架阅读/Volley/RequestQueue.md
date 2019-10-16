### **RequestQueue类**
> 请求队列是通过HashSet来实现存储的。

```
public void start() {
        stop(); //先调用stop()方法，必须确保当前所在运行的调度器都停止后才创建新的并开启
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();
        for (int i = 0; i < mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher =
                    new NetworkDispatcher(mNetworkQueue, mNetwork, mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }
其实CacheDispatcher和NetworkDispatcher都是继承自Thread类，是一个线程子类。
```

```
//调用quit()方法去停止CacheDispatcher 和 NetworkDispatcher
 public void stop() {
        if (mCacheDispatcher != null) {
            mCacheDispatcher.quit();
        }
        for (final NetworkDispatcher mDispatcher : mDispatchers) {
            if (mDispatcher != null) {
                mDispatcher.quit();
            }
        }
    }

```

```
//获取序列号
  public int getSequenceNumber() {
        return mSequenceGenerator.incrementAndGet();
    }
//获取缓存
 public Cache getCache() {
        return mCache;
    }
```

```
//接口类，用于RequestQueue的cancelAll时使用
 public interface RequestFilter {
        boolean apply(Request<?> request);
    }
```

```
 public void cancelAll(RequestFilter filter) {
        synchronized (mCurrentRequests) {
            for (Request<?> request : mCurrentRequests) {
                if (filter.apply(request)) {
                    request.cancel();
                }
            }
        }
    }
    
public void cancelAll(final Object tag) {
        if (tag == null) {
            throw new IllegalArgumentException("Cannot cancelAll with a null tag");//tag不能为空，否则会抛出异常。
        }
        cancelAll(
                new RequestFilter() {
                    @Override
                    public boolean apply(Request<?> request) {
                        return request.getTag() == tag;
                    }
                });
    }
 
```

```
 public <T> Request<T> add(Request<T> request) {
        request.setRequestQueue(this);//将request与requestqueue进行绑定
        synchronized (mCurrentRequests) {
            mCurrentRequests.add(request);//将request添加至hashset里
        }
        //设置序列号
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");
        sendRequestEvent(request, RequestEvent.REQUEST_QUEUED);
        //如果请求不可缓存，则跳过
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }
        mCacheQueue.add(request);
        return request;
    }

  <T> void finish(Request<T> request) {
         synchronized (mCurrentRequests) {
            mCurrentRequests.remove(request);//其实调用的是hashset.remove()
        }
        synchronized (mFinishedListeners) {
            for (RequestFinishedListener<T> listener : mFinishedListeners) {
                listener.onRequestFinished(request);//通过接口把状态回调出去
            }
        }
        //发送一个事件
        sendRequestEvent(request, RequestEvent.REQUEST_FINISHED);
    }
```

```
//发送生命周期给监听器
void sendRequestEvent(Request<?> request, @RequestEvent int event) {
        synchronized (mEventListeners) {
            for (RequestEventListener listener : mEventListeners) {
                listener.onRequestEvent(request, event);
            }
        }
    }

//给请求事件添加一个生命周期事件监听
public void addRequestEventListener(RequestEventListener listener) {
        synchronized (mEventListeners) {
            mEventListeners.add(listener);
        }
    }
//移除监听
public void removeRequestEventListener(RequestEventListener listener) {
        synchronized (mEventListeners) {
            mEventListeners.remove(listener);
        }
    }
    
@Deprecated
addRequestFinishedListener(RequestFinishedListener<T> listener)

@Deprecated
removeRequestFinishedListener(RequestFinishedListener<T> listener)
```