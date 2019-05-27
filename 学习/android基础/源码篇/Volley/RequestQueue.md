### **RequestQueue类**
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