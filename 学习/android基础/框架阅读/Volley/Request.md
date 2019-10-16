
**Request**类是一个抽象类，实现了Comparable接口




```
//默认采用utf-8字符集进行编码
private static final String DEFAULT_PARAMS_ENCODING = "UTF-8";
```


```
//支持多种请求方法，一般采用GET/POST居多
 public interface Method {
        int DEPRECATED_GET_OR_POST = -1;
        int GET = 0;
        int POST = 1;
        int PUT = 2;
        int DELETE = 3;
        int HEAD = 4;
        int OPTIONS = 5;
        int TRACE = 6;
        int PATCH = 7;
    }
```


```
interface NetworkRequestCompleteListener {

        //收到网络响应时的回调
        void onResponseReceived(Request<?> request, Response<?> response);

        //没有响应时的回调
        void onNoUsableResponseReceived(Request<?> request);
    }
```


```
//枚举类，标识请求的优先级
public enum Priority {
        LOW,
        NORMAL,
        HIGH,
        IMMEDIATE
    }
```

构造方法
```
 public Request(int method, String url, @Nullable Response.ErrorListener listener) {
        mMethod = method;
        mUrl = url;
        mErrorListener = listener;
        //设置重试策略
        setRetryPolicy(new DefaultRetryPolicy());

        //如果url是空的，则返回0，不为空则返回hashCode
        mDefaultTrafficStatsTag = findDefaultTrafficStatsTag(url);
    }
```


```
//获取method
public int getMethod() { return mMethod; }

//设置tag，setter采用链式结构
public Request<?> setTag(Object tag) {
        mTag = tag;
        return this;
    }
    
//获取tag
public Object getTag() {
        return mTag;
    }
    
//获取监听，可以返回空
public Response.ErrorListener getErrorListener() {
        synchronized (mLock) {
            return mErrorListener;
        }
    }

//获取传输状态标识    
public int getTrafficStatsTag() {
        return mDefaultTrafficStatsTag;
    }
    
//如果url是空的，则返回0，不为空则返回hashCode    
private static int findDefaultTrafficStatsTag(String url) {
        if (!TextUtils.isEmpty(url)) {
            Uri uri = Uri.parse(url);
            if (uri != null) {
                String host = uri.getHost();
                if (host != null) {
                    return host.hashCode();
                }
            }
        }
        return 0;
    }

//设置重试策略
public Request<?> setRetryPolicy(RetryPolicy retryPolicy) {
        mRetryPolicy = retryPolicy;
        return this;
    }
    
//添加时间日志，用于debug
public void addMarker(String tag) {
        if (MarkerLog.ENABLED) {
            mEventLog.add(tag, Thread.currentThread().getId());
        }
    }
```


```
void finish(final String tag) {
        if (mRequestQueue != null) {
            mRequestQueue.finish(this);//调用RequestQueue的finish方法去将请求remove调从而将request finish调
        }
        if (MarkerLog.ENABLED) {
            final long threadId = Thread.currentThread().getId();
            if (Looper.myLooper() != Looper.getMainLooper()) {
            //在主线程发送日志用于调试
                Handler mainThread = new Handler(Looper.getMainLooper());
                mainThread.post(
                        new Runnable() {
                            @Override
                            public void run() {
                                mEventLog.add(tag, threadId);
                                mEventLog.finish(Request.this.toString());
                            }
                        });
                return;
            }

            mEventLog.add(tag, threadId);
            mEventLog.finish(this.toString());
        }
    }
```

```
//调用RequestQueue的sendEvent方法发送事假
void sendEvent(@RequestQueue.RequestEvent int event) {
        if (mRequestQueue != null) {
            mRequestQueue.sendRequestEvent(this, event);
        }
    }

//设置序列号
public final Request<?> setSequence(int sequence) {
        mSequence = sequence;
        return this;
    }

//获取序列号
public final int getSequence() {
        if (mSequence == null) {
            throw new IllegalStateException("getSequence called before setSequence");
        }
        return mSequence;
    }

//获取url
public String getUrl() {
        return mUrl;
    }
    
//获取缓存的键值。先获取url和method，判断method是GET或者是默认的DEPRECATED_GET_OR_POST，直接返回url作为cacheKey，否则就返回Integer.toString(method) + '-' + url

public String getCacheKey() {
        String url = getUrl();
        int method = getMethod();
        if (method == Method.GET || method == Method.DEPRECATED_GET_OR_POST) {
            return url;
        }
        return Integer.toString(method) + '-' + url;
    }

//用于缓存一致性支持
public Request<?> setCacheEntry(Cache.Entry entry) {
        mCacheEntry = entry;
        return this;
    }
    
//可能会为空  
public Cache.Entry getCacheEntry() {
        return mCacheEntry;
    }

//用于标记请求已经取消    
public void cancel() {
        synchronized (mLock) {
            mCanceled = true;
            mErrorListener = null;
        }
    }

//如果请求已经被取消则返回true    
public boolean isCanceled() {
        synchronized (mLock) {
            return mCanceled;
        }
    }
    
//设置是否缓存此请求的响应
public final Request<?> setShouldCache(boolean shouldCache) {
        mShouldCache = shouldCache;
        return this;
    }
    
//获取是否设置缓存    
public final boolean shouldCache() {
        return mShouldCache;
    }

//设置服务器发生5XX 时是否需要重试    
public final Request<?> setShouldRetryServerErrors(boolean shouldRetryServerErrors) {
        mShouldRetryServerErrors = shouldRetryServerErrors;
        return this;
    }

//默认是NORMAL    
public Priority getPriority() {
        return Priority.NORMAL;
    }

//获取重试策略    
public RetryPolicy getRetryPolicy() {
        return mRetryPolicy;
    }
```

```
//设置RequestQueue，与RequestQueue进行绑定
public Request<?> setRequestQueue(RequestQueue requestQueue) {
        mRequestQueue = requestQueue;
        return this;
    }
```

```
//子类必须实现此方法去解析原始网络的请求，并返回合适的响应类型。
protected abstract Response<T> parseNetworkResponse(NetworkResponse response);

//子类能够覆写此方法去解析 网络错误，并返回一个更确切的错误。
protected VolleyError parseNetworkError(VolleyError volleyError) {
        return volleyError;
    }
 
//子类必须去实现此方法去将解析后的响应传递给它们的监听器   
protected abstract void deliverResponse(T response);

//将错误信息传递给ErrorListener
public void deliverError(VolleyError error) {
        Response.ErrorListener listener;
        synchronized (mLock) {
            listener = mErrorListener;
        }
        if (listener != null) {
            listener.onErrorResponse(error);
        }
    }
```


```
//实现comparable接口重写compareTo的目的是用于比较请求的优先级，如果是相同的优先级，则比较两个请求的序列号，否则比较枚举的常量序号
@Override
    public int compareTo(Request<T> other) {
        Priority left = this.getPriority();
        Priority right = other.getPriority();

        return left == right ? this.mSequence - other.mSequence : right.ordinal() - left.ordinal();
    }
```










