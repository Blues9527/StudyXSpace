**NetworkDispatcher类**
> NetworkDispatcher是继承自Thread类，所以NetworkDispatcher也是一个线程，其中的实现是通过BlockingQueue阻塞队列来实现调度的。

```
//使用BlockingQueue来进行存储请求
private final BlockingQueue<Request<?>> mQueue;

NetworkDispatcher(
            BlockingQueue<Request<?>> queue,
            Network network,
            Cache cache,
            ResponseDelivery delivery)//构造方法
            
//调用父类Thread.interrupt()
public void quit() {
        mQuit = true;
        interrupt();
    }

@Override
public void run() {
    //设置线程的优先级
    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
    
    //死循环，不断地区处理请求
    while (true) {
        try {
            processRequest();
        } catch (InterruptedException e) {
        //当发生异常时，如果已经退出了就调用interrupt去中断线程
            if (mQuit) {
                Thread.currentThread().interrupt();
                return;
            }
           ...
        }
        }
    }

//将请求从BlockingQueue中取出，调用另一个方法去处理
private void processRequest() throws InterruptedException {
        Request<?> request = mQueue.take();
        processRequest(request);
    }
    

void processRequest(Request<?> request) {
        long startTimeMs = SystemClock.elapsedRealtime();
        request.sendEvent(RequestQueue.RequestEvent.REQUEST_NETWORK_DISPATCH_STARTED);
        try {
            request.addMarker("network-queue-take");

            if (request.isCanceled()) {
                request.finish("network-discard-cancelled");
                request.notifyListenerResponseNotUsable();
                return;
            }

            addTrafficStatsTag(request);

            NetworkResponse networkResponse = mNetwork.performRequest(request);
            request.addMarker("network-http-complete");

            if (networkResponse.notModified && request.hasHadResponseDelivered()) {
                request.finish("not-modified");
                request.notifyListenerResponseNotUsable();
                return;
            }

            Response<?> response = request.parseNetworkResponse(networkResponse);
            request.addMarker("network-parse-complete");

            //如果可以缓存则写入缓存
            if (request.shouldCache() && response.cacheEntry != null) {
                mCache.put(request.getCacheKey(), response.cacheEntry);
                request.addMarker("network-cache-written");
            }

            request.markDelivered();
            //将请求回传过去
            mDelivery.postResponse(request, response);
            request.notifyListenerResponseReceived(response);
        } catch (VolleyError volleyError) {
            ...
            parseAndDeliverNetworkError(request, volleyError);
            request.notifyListenerResponseNotUsable();
        } catch (Exception e) {
            ...
            //将错误回传过去
            mDelivery.postError(request, volleyError);
            request.notifyListenerResponseNotUsable();
        } finally {
            request.sendEvent(RequestQueue.RequestEvent.REQUEST_NETWORK_DISPATCH_FINISHED);
        }
    }
```

**CacheDispatcher类**
```

public CacheDispatcher(
            BlockingQueue<Request<?>> cacheQueue,
            BlockingQueue<Request<?>> networkQueue,
            Cache cache,
            ResponseDelivery delivery) {
        mCacheQueue = cacheQueue;
        mNetworkQueue = networkQueue;
        mCache = cache;
        mDelivery = delivery;
        //管理正在等待的请求队列，通过相同的缓存key去删除多余的请求
        mWaitingRequestManager = new WaitingRequestManager(this);
    }
```

```
//调用父类的interrupt方法去中断
public void quit() {
        mQuit = true;
        interrupt();
    }
```

```
@Override
    public void run() {
        ...
        //设置线程优先级
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        //初始化缓存
        mCache.initialize();

        //通过不断循环去处理请求
        while (true) {
            try {
                processRequest();
            } catch (InterruptedException e) {
                //如果请求已经退出，也会终止当前线程并退出循环
                if (mQuit) {
                    Thread.currentThread().interrupt();
                    return;
                }
               ...
            }
        }
    }
```

```
private void processRequest() throws InterruptedException {
        final Request<?> request = mCacheQueue.take();
        processRequest(request);
    }
```


```
 void processRequest(final Request<?> request) throws InterruptedException {
        ...
        //发送声明周期事件
        request.sendEvent(RequestQueue.RequestEvent.REQUEST_CACHE_LOOKUP_STARTED);

        try {
            if (request.isCanceled()) {
            //如果请求被取消了，根据tag去结束掉改请求
                request.finish("cache-discard-canceled");
                return;
            }

            //根据缓存键从缓存中取出一个缓存项
            Cache.Entry entry = mCache.get(request.getCacheKey());
            
            //如果缓存项为空，给请求添加一个miss的标记，return
            if (entry == null) {
                request.addMarker("cache-miss");
                if (!mWaitingRequestManager.maybeAddToWaitingRequests(request)) {
                    mNetworkQueue.put(request);
                }
                return;
            }
            //如果缓存项过期了，则重新设置缓存项，return
            if (entry.isExpired()) {
                request.addMarker("cache-hit-expired");
                request.setCacheEntry(entry);
                 if (!mWaitingRequestManager.maybeAddToWaitingRequests(request)) {
                    mNetworkQueue.put(request);
                }
                return;
            }

            //如果缓存项既不为空也没有过期，则通过parseNetworkResponse去解析得到一个response
            request.addMarker("cache-hit");
            Response<?> response =
                    request.parseNetworkResponse(
                            new NetworkResponse(entry.data, entry.responseHeaders));
            request.addMarker("cache-hit-parsed");

            if (!entry.refreshNeeded()) {
            //如果缓存项不需要刷新，则直接将response post出去
                mDelivery.postResponse(request, response);
            } else {
                request.addMarker("cache-hit-refresh-needed");
                //重新设置缓存项
                request.setCacheEntry(entry);
                将response的intermediate属性设置为true
                response.intermediate = true;

                if (!mWaitingRequestManager.maybeAddToWaitingRequests(request)) {
                    mDelivery.postResponse(
                            request,
                            response,
                            new Runnable() {
                                @Override
                                public void run() {
                                    try {
                                        mNetworkQueue.put(request);
                                    } catch (InterruptedException e) {
                                        Thread.currentThread().interrupt();
                                    }
                                }
                            });
                } else {
                    mDelivery.postResponse(request, response);
                }
            }
        } finally {
            request.sendEvent(RequestQueue.RequestEvent.REQUEST_CACHE_LOOKUP_FINISHED);
        }
    }
```


```
WaitingRequestManager类是CacheDispatcher的一个静态内部类，实现了Request的NetworkRequestCompleteListener接口，主要的目的是：实现网络请求完成的监听，管理等待请求列表和使用相同的缓存键删除重复请求

//使用hashMap去存储等待的请求
private final Map<String, List<Request<?>>> mWaitingRequests = new HashMap<>();

//与CacheDispatcher进行绑定
private final CacheDispatcher mCacheDispatcher;

WaitingRequestManager(CacheDispatcher cacheDispatcher) {
            mCacheDispatcher = cacheDispatcher;
        }
        

 @Override
        public void onResponseReceived(Request<?> request, Response<?> response) {
        //如果response里没有缓存项，将request回调到onNoUsableResponseReceived，然后return
            if (response.cacheEntry == null || response.cacheEntry.isExpired()) {
                onNoUsableResponseReceived(request);
                return;
            }
            
            String cacheKey = request.getCacheKey();
            List<Request<?>> waitingRequests;
            synchronized (this) {
                waitingRequests = mWaitingRequests.remove(cacheKey);
            }
            if (waitingRequests != null) {
                ...
                for (Request<?> waiting : waitingRequests) {
                    mCacheDispatcher.mDelivery.postResponse(waiting, response);
                }
            }
        }
        
@Override
public synchronized void onNoUsableResponseReceived(Request<?> request) {
    String cacheKey = request.getCacheKey();
    List<Request<?>> waitingRequests = mWaitingRequests.remove(cacheKey);
    if (waitingRequests != null && !waitingRequests.isEmpty()) {
        ...
        Request<?> nextInLine = waitingRequests.remove(0);
        mWaitingRequests.put(cacheKey, waitingRequests);
        nextInLine.setNetworkRequestCompleteListener(this);
        try {
            mCacheDispatcher.mNetworkQueue.put(nextInLine);
        } catch (InterruptedException iex) {
            VolleyLog.e("Couldn't add request to queue. %s", iex.toString());
            // Restore the interrupted status of the calling thread (i.e. NetworkDispatcher)
            Thread.currentThread().interrupt();
            // Quit the current CacheDispatcher thread.
            mCacheDispatcher.quit();
        }
    }
}

private synchronized boolean maybeAddToWaitingRequests(Request<?> request) {

    //通过传进来的请求去获取缓存key
    String cacheKey = request.getCacheKey();
    //判断mWaitingRequests是否包含该缓存键
    if (mWaitingRequests.containsKey(cacheKey)) {
        //通过该缓存键去取出对应的请求列表
        List<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
        //判断列表是否为空
        if (stagedRequests == null) {
            //如果为空则实例化一个arraylist
            stagedRequests = new ArrayList<>();
        }
        request.addMarker("waiting-for-response");
        //确保缓存键里有对应的请求队列
        stagedRequests.add(request);
        mWaitingRequests.put(cacheKey, stagedRequests);
        ...
        return true;
    } else {
        //如果mWaitingRequests里没有该缓存键，则存一个null的list进去，并return false
        mWaitingRequests.put(cacheKey, null);
        request.setNetworkRequestCompleteListener(this);
        ...
        return false;
    }
}
```





