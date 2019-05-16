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
顺着使用方法主线看下去，先看**Volley类**
> 创建RequestQueue实例的时候，是通过Volley类中的newRequestQueue()方法来获取的。其中，Volley类中是空构造，newRequestQueue()方法是静态方法，所以可以直接new Volley().newRequestQueue()。

> 其中newRequestQueue()利用了方法重载，总共有四个方法，但是最终调用的都是私有的RequestQueue newRequestQueue(Context context, Network network)方法。而其它三个public方法中RequestQueue newRequestQueue(Context context, HttpStack stack)方法已经被弃用，因为HttpStack被弃用了。

> newRequestQueue()方法里面会实例化一个RequestQueue
```
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network)
```
> 最后会调用RequestQueue的start()方法，Volley类到此就结束了。

然后再看**RequestQueue类**
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
接着再看**Request类**






