# Volley的简单使用

```
以StringRequest为例：

1.构建一个请求队列
  RequestQueue rq = new Volley().newRequestQueue(mContext);
  
2.构建一个请求体
  StringRequest sr = new StringRequest(Request.Method.GET, url, new       Response.Listener<String>() {
        @Override
        public void onResponse(String response) {

        }
    }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {

        }
    });
3.将请求体放入请求队列
    rq.add(sr);
```


## 分析
### 顺着使用方法主线看下去，先看**Volley类**
> 创建RequestQueue实例的时候，是通过Volley类中的newRequestQueue()方法来获取的。其中，Volley类中是空构造，newRequestQueue()方法是静态方法，所以可以直接new Volley().newRequestQueue()。

> 其中newRequestQueue()利用了方法重载，总共有四个方法，但是最终调用的都是私有的RequestQueue newRequestQueue(Context context, Network network)方法。而其它三个public方法中RequestQueue newRequestQueue(Context context, HttpStack stack)方法已经被弃用，因为HttpStack被弃用了。

> newRequestQueue()方法里面会实例化一个RequestQueue
```
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network)
```
> 最后会调用RequestQueue的start()方法，Volley类到此就结束了。

### 然后再看**RequestQueue类**
> 请求队列是通过HashSet来实现存储的。

```
start()方法：先调用stop()方法，必须确保当前所在运行的调度器都停止后才创建新的并开启。然后开启CacheDispatcher 和 NetworkDispatcher；

其实CacheDispatcher和NetworkDispatcher都是继承自Thread类，是一个线程子类。
```

```
stop()方法：调用quit()方法去停止CacheDispatcher 和 NetworkDispatcher
```

```
getSequenceNumber()：获取序列号
getCache()：获取缓存
```

```
RequestFilter 接口类，用于RequestQueue的cancelAll时使用
```

```
 cancelAll(RequestFilter filter)
 cancelAll(final Object tag)：tag不能为空，否则会抛出异常。
 
 cancelAll()方法具有方法重载的特性
```

```
add(Request<T> request)方法：将请求添加到调度队列（hashset.add()）
finish(Request<T> request)方法:结束掉请求(hashset.remove())
```

```
sendRequestEvent(Request<?> request, @RequestEvent int event) 

addRequestEventListener(RequestEventListener listener)

removeRequestEventListener(RequestEventListener listener) 

addRequestFinishedListener(RequestFinishedListener<T> listener)

removeRequestFinishedListener(RequestFinishedListener<T> listener)
```
### 接着再看**Request类**
> Reuqest类是一个抽象类，实现了Comparable接口

构造函数

```
@Deprecated
Request(String url, Response.ErrorListener listener)
此构造方法是调用下方的方法的，默认的method是Method.DEPRECATED_GET_OR_POST

Request(int method, String url, @Nullable Response.ErrorListener listener)

下方的构造方法中会调用setRetryPolicy（RetryPolicy retryPolicy）传入的是实现了RetryPolicy接口的空构造方法DefaultRetryPolicy()

```

主要方法

```
getMethod()//获取构造函数传进来或者默认的method

setTag(Object tag)/getTag()//设置tag，可以通过tag去取消某个请求

getErrorListener()//获取错误监听器，带同步锁

setRetryPolicy(RetryPolicy retryPolicy)//设置重试机制

finish(final String tag)//通知请求已经结束（成功或者错误）

sendEvent(@RequestQueue.RequestEvent int event)//发送事件

setRequestQueue(RequestQueue requestQueue)//将请求与队列关联起来

setSequence(int sequence)/ getSequence()//设置/获取序列号

getUrl()//获取url

cancel()/标记请求被取消

isCanceled()//判断请求是否被取消
```

### Request类是一个抽象类，接下来看一下Request的实现类
> Request的实现类有5个，分别是StringReqeust、JsonRequest（抽象类）、JsonObjectRequest、JsonArrayRequest、ImageRequest

JsonRequest类
> 继承自Request类，也是一个抽象类，同时也是JsonArrayRequest和JsonObjectRequest的父类。

> 注意一点，重写parseNetworkResponse方法也并没有具体实现，还是abstract。

> 对比Request，JsonRequest多传入了一个Response.Listener监听，在deliverResponse方法里将response回调出去。

```
@Deprecated
JsonRequest(
            String url, 
            String requestBody, 
            Listener<T> listener, 
            ErrorListener errorListener)

JsonRequest(
            int method,
            String url,
            @Nullable String requestBody,
            Listener<T> listener,
            @Nullable ErrorListener errorListener)
            
上面的方法调用了下方的方法，传入method参数Method.DEPRECATED_GET_OR_POST
```

JsonObjectRequest类

```
JsonObjectRequest(
            int method,
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener)
            
JsonObjectRequest(
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener)
下面的方法调用了上面的方法，传进去method jsonRequest == null ? Method.GET : Method.POST

最终都是调用父类的方法，只不过限定了Listener的传入类型为JSONObject

```

JsonArrayRequest类

```
 JsonArrayRequest(
            String url, 
            Listener<JSONArray> listener, 
            @Nullable ErrorListener errorListener)
调用父类方法，传入默认method为Method.GET，requestBody为null
            
JsonArrayRequest(
            int method,
            String url,
            @Nullable JSONArray jsonRequest,
            Listener<JSONArray> listener,
            @Nullable ErrorListener errorListener)
调用父类方法，传入requestBody (jsonRequest == null) ? null : jsonRequest.toString()
```

ImageRequest类
> 默认超时时间为 1s

> 默认最大重试次数为2

> 限定传进来的Listener的类型为Bitmap

> 通过锁去限制每次解码不超过一张图片

```
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            ScaleType scaleType,
            Config decodeConfig,
            @Nullable Response.ErrorListener errorListener)
默认传入method为Method.GET

@Deprecated
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            Config decodeConfig,
            Response.ErrorListener errorListener)
调用上方方法，传入默认配置ScaleType.CENTER_INSIDE

getPriority()//return Priority.LOW

getResizedDimension(
            int maxPrimary,
            int maxSecondary,
            int actualPrimary,
            int actualSecondary,
            ScaleType scaleType)
            
doParse(NetworkResponse response)//核心方法，调用此方法解析出来一个bitmap并通过Response.success/error将结果传递出去。
```

StringRequest类

```
StringRequest(
            int method,
            String url,
            Listener<String> listener,
            @Nullable ErrorListener errorListener)
//调用父类犯法，传入限定String型的Listener
            
StringRequest(
            String url, Listener<String> listener, @Nullable ErrorListener errorListener)
//调用上面方法，传入默认的method Method.GET
```



RetryPolicy类

```
getCurrentTimeout()//获取当前超时时间

getCurrentRetryCount()//获取当前重试次数

retry(VolleyError error)//重试
```

NetworkDispatcher类
> 上面说到，NetworkDispatcher是继承自Thread类，所以NetworkDispatcher也是一个线程，其中的实现是通过阻塞队列来实现调度的。


```
NetworkDispatcher(
            BlockingQueue<Request<?>> queue,
            Network network,
            Cache cache,
            ResponseDelivery delivery)//构造方法
            
quit()//调用Thread.interrupt()

run()//覆写Thread类的run方法
```









