# ThreadPool(线程池)

    1.1）常用的功能线程池
            1）FixedThreadPool（定长线程池）
            2）ScheduleThreadPool（定时线程池）
            3）CachedThreadPool（可缓存线程池）
            4）SingleThreadExecutor（单线程化线程池）
    1.2）使用流程
            1）创建线程池，配置线程池参数
                Executor threadPool = ew ThreadPoolExecutor(参数...);
            2)向线程池提交任务，调用 execute()方法，传入 Runnable对象
                threadPool.execute(Runnable run);
            3使用完之后需要关闭线程池,可调用shutdown()方法
                threadPool.shutdown();
                
                
```
//用法
    Executors.newCachedThreadPool().execute(new Runnable() {
            @Override
            public void run() {

            }
        });
        
//用法
     Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                
            }
        });
        ExecutorService pool = Executors.newCachedThreadPool();
        pool.execute(t);
```

分析：

ThreadPoolExecutor ->ExecutorService -> AbstractExecutorService ->Executor

==**Executor**是一个接口类==

```
 void execute(Runnable command);
```

==**ExecutorService**是继承了Executor的一个接口类==

```
void shutdown();//有序地关闭，如果是已启用的话调用shut down是没有效果的，但是后续要执行的就会被shut down

List<Runnable> shutdownNow();//此方法不会等待所有正在执行的任务的终止，最终会返回一个未执行任务列表

boolean isShutdown();//如果当前executor已经被shutdown则返回true

boolean isTerminated();//如果所有任务都完成后并终止则返回true，前提条件是必须要有调用过shutdown 或者是 shutdownNow方法。

boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;//如果executor已经终止则返回true，如果executor在终止之前就已经超时了则返回false
        
//Future表示任务正在等待处理的结果，可以通过Future.get方法返回任务的结果
<T> Future<T> submit(Callable<T> task);

<T> Future<T> submit(Runnable task, T result);

Future<?> submit(Runnable task);

<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
        
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
        
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
        
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

==**isShutdown()与isTerminated()的区别：**==

> isShutDown当调用shutdown()或shutdownNow()方法后返回为true。

> isTerminated当调用shutdown()方法后，并且所有提交的任务完成后返回为true;

> isTerminated当调用shutdownNow()方法后，成功停止后返回为true;

> 如果线程池任务正常完成，都为false

==**Future**类是一个接口类，具体存储数据的都是Future的实现类==

```
boolean cancel(boolean mayInterruptIfRunning);//尝试去取消一个任务。如果任务已经完成、已经被取消了或者是由于某些原因不能被取消，可能会存在失败的情况。当改方法返回true的时候，isDone()和isCancelled方法都会一直返回true

boolean isCancelled();//如果任务在其正常完成前被取消了则返回true

boolean isDone();//如果任务完成则返回true

V get() throws InterruptedException, ExecutionException;//必要时等待计算结束，然后返回检索的结果

V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

==**AbstractExecutorService**是一个抽象类，实现了ExecutorService接口==

```
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
    
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
    
//从0开始，取消全部
private static <T> void cancelAll(ArrayList<Future<T>> futures) {
        cancelAll(futures, 0);
    }

//从给定的j开始，取消到结束
private static <T> void cancelAll(ArrayList<Future<T>> futures, int j) {
        for (int size = futures.size(); j < size; j++)
            futures.get(j).cancel(true);
    }
```

==**ThreadPoolExecutor**继承自AbstractExecutorService==

```
@params corePoolSize:保存在池中的线程数量，即使是处于闲置状态
@params maximumPoolSize:池中允许的最大线程数量
@params keepAliveTime：多余空闲线程在终止前等待新任务的最长你时间
@params unit：keepAliveTime的时间单位
@params workQueue：保存未执行的通过execute提交并实现了runnable接口任务。
@params threadFactory：创建新的线程用的
@params handler：当达到了线程的边界和队列的容量时导致程序阻塞时使用的handler(拒绝策略)
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```


```
 public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```






