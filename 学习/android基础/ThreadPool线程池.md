# ThreadPool(线程池)
[参考文章](https://www.imooc.com/article/23728?block_id=tuijian_wz)

        2.1）常用的功能线程池
            1）FixedThreadPool（定长线程池）
            2）ScheduleThreadPool（定时线程池）
            3）CachedThreadPool（可缓存线程池）
            4）SingleThreadExecutor（单线程化线程池）
        2.2）使用流程
            1）创建线程池，配置线程池参数
                Executor threadPool = new ThreadPoolExecutor(参数...);
            2)向线程池提交任务，调用 execute()方法，传入 Runnable对象
                threadPool.execute(Runnable run);
            3)使用完之后需要关闭线程池,可调用shutdown()方法
                threadPool.shutdown();
                
                
```
//用法
    Executors.newCachedThreadPool().execute(new Runnable() {
            @Override
            public void run() {

            }
        });
```
