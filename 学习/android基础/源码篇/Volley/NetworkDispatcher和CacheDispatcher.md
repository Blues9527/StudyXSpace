NetworkDispatcher类
> NetworkDispatcher是继承自Thread类，所以NetworkDispatcher也是一个线程，其中的实现是通过BlockingQueue阻塞队列来实现调度的。

```
NetworkDispatcher(
            BlockingQueue<Request<?>> queue,
            Network network,
            Cache cache,
            ResponseDelivery delivery)//构造方法
            
quit()//调用Thread.interrupt()

run()//覆写Thread类的run方法
```