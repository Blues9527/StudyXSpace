[toc]

# Kotlin Flow
> Flow库是在Kotlin Coroutines 1.3.2 新增的库。

[Flow操作符大全](https://juejin.cn/post/6989536876096913439)

## 冷流
> 指普通的`flow`，其具体实现是`SafeFlow`，一般通过`flow{}`创建，而冷流一般指的是只有被`collect`的时候才会`emit`数据，并且每次`collect`都会调用`emit`。


```kotlin
//举个例子，但一般不会这样用
val flow = flow{
    emit("hello flow~")
}

flow.collect {
    Log.i("Blues","collect1 flow:$it")
}

flow.collect {
    Log.i("Blues","collect2 flow:$it")
}

//输出
// collect1 flow:hello flow~
// collect2 flow:hello flow~
```

### 实现逻辑
#### 第一步: 通过`flow{}`创建`Flow`

```kotlin
//Builders.kt
public fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)

private class SafeFlow<T>(private val block: suspend FlowCollector<T>.() -> Unit) : AbstractFlow<T>() {
    override suspend fun collectSafely(collector: FlowCollector<T>) {
        collector.block()
    }
}
```

#### 第二步: `collect`
```kotlin
//Collect.kt
public suspend inline fun <T> Flow<T>.collect(crossinline action: suspend (value: T) -> Unit): Unit =
    collect(object : FlowCollector<T> {
        override suspend fun emit(value: T) = action(value)
    })

```
> 上述`collect`方法会调用到`SafeFlow`的`collect`方法，但是`SafeFlow`没有实现该方法，就看父类`AbstractFlow`的`collect`方法

```kotlin
//Flow.kt
public abstract class AbstractFlow<T> : Flow<T>, CancellableFlow<T> {

    @InternalCoroutinesApi
    public final override suspend fun collect(collector: FlowCollector<T>) {
        val safeCollector = SafeCollector(collector, coroutineContext)
        try {
            collectSafely(safeCollector)
        } finally {
            safeCollector.releaseIntercepted()
        }
    }

    public abstract suspend fun collectSafely(collector: FlowCollector<T>)
}

```

> 这也就解释了为什么冷流需要调用`collect`才会`emit`数据。


#### 第三步: emit

```kotlin
//SafeCollector.kt
override suspend fun emit(value: T) {
    return suspendCoroutineUninterceptedOrReturn sc@{ uCont ->
        try {
            emit(uCont, value)
        } catch (e: Throwable) {
            //...
        }
    }
}

private fun emit(uCont: Continuation<Unit>, value: T): Any? {

    //判断两个协程上下文是否一致
    val currentContext = uCont.context
    currentContext.ensureActive()
    val previousContext = lastEmissionContext
    if (previousContext !== currentContext) {
        checkContext(currentContext, previousContext, value)
    }
    completion = uCont
    
    //emitFun目的是统一Continuation
    return emitFun(collector as FlowCollector<Any?>, value, this as Continuation<Unit>)
}

private val emitFun =
    FlowCollector<Any?>::emit as Function3<FlowCollector<Any?>, Any?, Continuation<Unit>, Any?>
```
> `emitFun`这里比较抽象，最终调用的是`FlowCollector`的`emit()`方法。反编译成Java查看一下

```Java
private static final Function3 emitFun = (Function3)TypeIntrinsics.beforeCheckcastToFunctionOfArity(new Function3() {
  // $FF: synthetic method
  // $FF: bridge method
  public Object invoke(Object var1, Object var2, Object var3) {
    //调用下方的invoke方法
     return this.invoke((FlowCollector)var1, var2, (Continuation)var3);
  }

  @Nullable
  public final Object invoke(@NotNull FlowCollector p1, @Nullable Object p2, @NotNull Continuation continuation) {
    //这里的collector是构造函数传进来的collector，就是collect(FlowCollector)传进来的collector
     return p1.emit(p2, continuation);
  }
}, 3);
```


## 热流
> `SharedFlow`和`StateFlow`，`collector`收集数据之前已经将数据发送出去


### SharedFlow
> 构建`SharedFlow`有两种方式，一种是通过构建`MutableSharedFlow`，然后手动`emit`；另一种是通过`Flow.shareIn()`，然后直接`collect`。

```kotlin
//举个例子，一般不会这样写
val sharedFlow = MutableSharedFlow<Int>(replay = 2)// 1.构建 SharedFlow

repeat(5){
    sharedFlow.emit(it) //2.发射数据流
}

sharedFlow.collect { //3.接收数据流
    Log.i("Blues", "share flow collect:$it")
}

//输出结果受replay参数影响。如果是0，则没有输出
//share flow collect:3
//share flow collect:4

------------------------------------------

//举个例子，一般这样写比较多
private suspend fun testShareFlow(): SharedFlow<Int> {

    return flowOf(1, 2, 3).shareIn(
        CoroutineScope(Dispatchers.IO),
        SharingStarted.WhileSubscribed(),
        2
    )
}

testShareFlow().collect {
    Log.i("Blues", "share flow by shareIn collect  it:$it")
}

//输出
//share flow by shareIn collect  it:1
//share flow by shareIn collect  it:2
//share flow by shareIn collect  it:3

```

#### 第一种: `MutableSharedFlow`方式实现逻辑
##### 创建`MutableSharedFlow`

```kotlin
//ShareFlow.kt

//这里的方法名和接口名一样。。。
public fun <T> MutableSharedFlow(
    replay: Int = 0, //新订阅者能接收到值的数量
    extraBufferCapacity: Int = 0, //除replay外额外缓冲值的数量
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND //缓冲区溢流策略
): MutableSharedFlow<T> {
    
    //缓存区容量= replay + extraBufferCapacity
    val bufferCapacity0 = replay + extraBufferCapacity
    val bufferCapacity = if (bufferCapacity0 < 0) Int.MAX_VALUE else bufferCapacity0 // coerce to MAX_VALUE on overflow
    
    //创建一个SharedFlowImpl
    return SharedFlowImpl(replay, bufferCapacity, onBufferOverflow)
}

//溢流时挂起
BufferOverflow#SUSPEND
//溢流时丢弃最老的值
BufferOverflow#DROP_OLDEST
//溢流时丢弃最新的值
BufferOverflow#DROP_LATEST
```
![image](https://note.youdao.com/yws/api/personal/file/WEB7472d06d418def1f875f106bb8d67e64?method=download&shareKey=6338d159ed1beff0e8d790d40f133da5)


##### `emit`实现

```kotlin
override suspend fun emit(value: T) {
    if (tryEmit(value)) return
    emitSuspend(value)
}

override fun tryEmit(value: T): Boolean {
    var resumes: Array<Continuation<Unit>?> = EMPTY_RESUMES
    //这里也是true
    val emitted = synchronized(this) {
        //这里是true
        if (tryEmitLocked(value)) {
            resumes = findSlotsToResumeLocked(resumes)
            true
        } else {
            false
        }
    }
    for (cont in resumes) cont?.resume(Unit)
    return emitted
}

private fun tryEmitLocked(value: T): Boolean {
    
    //无收集者，能使nCollectors不为0的唯一办法就是调用collect。所以有无搜集者的判断方式是collect在前还是emit在前
    if (nCollectors == 0) return tryEmitNoCollectorsLocked(value) 
    //....
    
    //当buffer的size >= buffer的容量且最小的收集者下标<=replay下标的时候会触发溢流策略处理
    if (bufferSize >= bufferCapacity && minCollectorIndex <= replayIndex) {
        when (onBufferOverflow) {
            BufferOverflow.SUSPEND -> return false //挂起
            BufferOverflow.DROP_LATEST -> return true //丢弃最新的
            BufferOverflow.DROP_OLDEST -> {} // 强制入队并丢弃最老的
        }
    }
    enqueueLocked(value)
    bufferSize++
    if (bufferSize > bufferCapacity) dropOldestLocked()
    //...
}

private fun tryEmitNoCollectorsLocked(value: T): Boolean {
    if (replay == 0) return true // 如果replay为0，则不需要后续操作
    enqueueLocked(value) //将value入队到 replayCache
    bufferSize++ //这里表示值已添加到buffer里
    //判断是否需要丢弃最老的value
    if (bufferSize > replay) dropOldestLocked()
    //更新minCollectorIndex
    minCollectorIndex = head + bufferSize
    return true
}

private fun dropOldestLocked() {
    //这里是将头部index item置空
    buffer!!.setBufferAt(head, null)
    //...
}

//将值插入array
private fun enqueueLocked(item: Any?) {
    val curSize = totalSize
    //扩展buffer array
    val buffer = when (val curBuffer = buffer) {
        null -> growBuffer(null, 0, 2)
        else -> if (curSize >= curBuffer.size) growBuffer(curBuffer, curSize,curBuffer.size * 2) else curBuffer//每次扩容都是2倍的当前容量
    }
    //在 head + curSize & size-1处插入 value
    //如果是首次，head = minOf(minCollectorIndex, replayIndex)，其中minCollectorIndex =0，replayIndex=0
    buffer.setBufferAt(head + curSize, item)
}
//这里举个例子，就是首次的时候会创建一个长度为2的array，然后插入的下标为head=0，curSize=0，size-1 =1 ，就是 00 & 01 = 00，就是在0的位置插入value
```
![image](https://note.youdao.com/yws/api/personal/file/WEB7a2597e93c7d7f54451197fea96c2184?method=download&shareKey=1f436a044eb79926271d182195894a47)


```kotlin
//溢流时触发的操作
    private suspend fun emitSuspend(value: T) = suspendCancellableCoroutine<Unit> sc@{ cont ->
        //...
        val emitter = synchronized(this) lock@{
            //再次检查buffer是不是满了
            if (tryEmitLocked(value)) {
                cont.resume(Unit)
                resumes = findSlotsToResumeLocked(resumes)
                return@lock null
            }
            //创建emitter塞入队列
            Emitter(this, head + totalSize, value, cont).also {
                enqueueLocked(it)
                queueSize++ //用于计算队列的大小
                //...
            }
        }
        //...
    }
```


##### collect实现

```kotlin
public suspend inline fun <T> Flow<T>.collect(crossinline action: suspend (value: T) -> Unit): Unit =
    collect(object : FlowCollector<T> {
        override suspend fun emit(value: T) = action(value)
    })

//SharedFlow.kt
override suspend fun collect(collector: FlowCollector<T>) {
    //分配空槽，并且nCollectors++
    val slot = allocateSlot()
    try {
        if (collector is SubscribedFlowCollector) collector.onSubscription()
        val collectorJob = currentCoroutineContext()[Job]
        while (true) {
            var newValue: Any?
            while (true) {
                newValue = tryTakeValue(slot)
                if (newValue !== NO_VALUE) break
                awaitValue(slot)
            }
            collectorJob?.ensureActive()
            //调用collector的emit
            collector.emit(newValue as T)
        }
    } finally {
        //collect完释放slot
        freeSlot(slot)
    }
}

private fun tryTakeValue(slot: SharedFlowSlot): Any? {
    var resumes: Array<Continuation<Unit>?> = EMPTY_RESUMES
    val value = synchronized(this) {
        val index = tryPeekLocked(slot)
        if (index < 0) {
            NO_VALUE
        } else {
            val oldIndex = slot.index
            val newValue = getPeekedValueLockedAt(index)
            slot.index = index + 1
            resumes = updateCollectorIndexLocked(oldIndex)
            newValue
        }
    }
    //...
    return value
}

private fun getPeekedValueLockedAt(index: Long): Any? =
    //这里也是index & size-1
    when (val item = buffer!!.getBufferAt(index)) {
        is Emitter -> item.value
        else -> item
    }
```

#### 第二种: Flow.shareIn()实现逻辑
> 冷流转热流

##### flowOf()

```kotlin
//Builders.kt
public fun <T> flowOf(vararg elements: T): Flow<T> = flow {
    for (element in elements) {
        emit(element)
    }
}

//上述的flow的实现就是这个unsafeFlow
internal inline fun <T> unsafeFlow(@BuilderInference crossinline block: suspend FlowCollector<T>.() -> Unit): Flow<T> {
    return object : Flow<T> {
        override suspend fun collect(collector: FlowCollector<T>) {
            collector.block()
        }
    }
}

//这里就是创建一个Flow，然后重写collect。这里就是调用collect就是将值发射出去
```

##### shareIn()

SharingStarted策略
- SharingStarted.WhileSubscribed()
> 当存在活跃订阅者时，开始共享数据

- SharingStarted.Eagerly
> 可立即启动提供方为订阅者提供数据

- SharingStarted.Lazily
> 在第一个订阅者出现后开始共享数据，并使数据流永远保持活跃状态

```kotlin
//Share.kt
public fun <T> Flow<T>.shareIn(
    scope: CoroutineScope,
    started: SharingStarted,
    replay: Int = 0
): SharedFlow<T> {
    
    //创建默认配置
    val config = configureSharing(replay)
    //创建MutableSharedFlow
    val shared = MutableSharedFlow<T>(
        replay = replay,
        extraBufferCapacity = config.extraBufferCapacity,
        onBufferOverflow = config.onBufferOverflow
    )
    
    //这里是启动协程，然后执行upstream.collect()
    val job = scope.launchSharing(config.context, config.upstream, shared, started, NO_VALUE as T)
    return ReadonlySharedFlow(shared, job)
}
```

```kotlin
private fun <T> Flow<T>.configureSharing(replay: Int): SharingConfig<T> {
    
    //计算extra的容量
    val defaultExtraCapacity = replay.coerceAtLeast(Channel.CHANNEL_DEFAULT_CAPACITY) - replay
    
    //...
    
    return SharingConfig(
        upstream = this,//就是flowOf创建的unsafeFlow
        extraBufferCapacity = defaultExtraCapacity,
        onBufferOverflow = BufferOverflow.SUSPEND,
        context = EmptyCoroutineContext
    )
}
```


```kotlin
private fun <T> CoroutineScope.launchSharing(
    context: CoroutineContext,
    upstream: Flow<T>,
    shared: MutableSharedFlow<T>,
    started: SharingStarted,
    initialValue: T
): Job =
    launch(context) { 
        when {
            started === SharingStarted.Eagerly -> {
            //这里的upstream就是unsafeFlow，所以这里执行的是emit循环操作
                upstream.collect(shared)
            }
            started === SharingStarted.Lazily -> {
                //这个first是一个挂起函数，不满足时挂起等待
                shared.subscriptionCount.first { it > 0 }
                upstream.collect(shared)
            }
            else -> {
               //自定义策略，其实最终也会调SharingCommand.START
               started.command(shared.subscriptionCount)
                    .distinctUntilChanged()
                    .collectLatest {
                        when (it) {
                            SharingCommand.START -> upstream.collect(shared) // can be cancelled
                            SharingCommand.STOP -> { //取消、不做处理 }
                            SharingCommand.STOP_AND_RESET_REPLAY_CACHE -> {
                                if (initialValue === NO_VALUE) {
                                    shared.resetReplayCache() 
                                } else {
                                    shared.tryEmit(initialValue)
                                }
                            }
                        }
                    }
            }
        }
    }

```

##### collect()

```kotlin
//Collect.kt

//这里的action就是那个log代码块
//collect走的SharedFlowImpl的collect。这里的操作就是创建完ReadonlySharedFlow后立即执行SharedFlow#collect()，上述的emit是挂起函数，执行collect会出现buffer是空的情况，这时候走的就是挂起循环等待值的流程
public suspend inline fun <T> Flow<T>.collect(crossinline action: suspend (value: T) -> Unit): Unit =
    collect(object : FlowCollector<T> {
        override suspend fun emit(value: T) = action(value)
    })

override suspend fun collect(collector: FlowCollector<T>) {

    val slot = allocateSlot()
    //...
    while (true) {
        var newValue: Any?
        while (true) {
            newValue = tryTakeValue(slot)
            if (newValue !== NO_VALUE) break
            awaitValue(slot) //走到这里，挂起等待
        }
        //..然后调用打印日志的block
        collector.emit(newValue as T)
    }
}
```


#### Slot
> 存储格式是`Array`，创建&扩容方式和`buffer`一样。首次会创建`size=2`的`Array`，然后会创建一个`slot`放入`Array`头部


```kotlin
//创建slots array
val slots = when (val curSlots = slots) {
    null -> createSlotArray(2).also { slots = it }
    else -> if (nCollectors >= curSlots.size) {
        curSlots.copyOf(2 * curSlots.size).also { slots = it }
    } else {
        curSlots
    }
}
            
//
var index = nextIndex
var slot: S
while (true) {
    //创建slot并设置进slots array
    slot = slots[index] ?: createSlot().also { slots[index] = it }
    index++
    if (index >= slots.size) index = 0
    if ((slot as AbstractSharedFlowSlot<Any>).allocateLocked(this)) break //找到可分配的 空闲 slot 就break
}
```


### StateFlow
> `StateFlow`和`SharedFlow`相似，因为`StateFlow`是继承自`SharedFlow`的。构建`StateFlow`有两种方式，一种是通过构建`MutableStateFlow`，然后手动`emit`；另一种是通过`Flow.stateIn()`，然后直接`collect`。

```kotlin
//举个例子，一般不会这样写
val stateFlow = MutableStateFlow(-1)

repeat(5) {
    stateFlow.emit(it)
}

stateFlow.collect {
    Log.i("Blues", "state flow by MutableStateFlow collect:$it")
}

//输出最后一次发射的内容，如果没有手动调用emit就发送默认值
//state flow by MutableStateFlow collect:4

-----------------------------------------------

//举个例子，一般都这样写
private suspend fun testStateFlow(): StateFlow<Int> {
    return flowOf(1, 2, 3).stateIn(
        CoroutineScope(Dispatchers.IO),
        SharingStarted.WhileSubscribed(),
        -1
    )
}

testStateFlow().collect {
    Log.i("Blues", "state flow by stateIn collect it:$it")
}

//输出
//state flow by stateIn collect it:-1
//state flow by stateIn collect it:3
```

#### 第一种: MutableStateFlow实现逻辑

#####  创建`MutableStateFlow`
> 内部通过成员变量`value`来存储值

```kotlin
//StateFlow.kt

//可以看出，StateFlow是有默认值的。而SharedFlow默认值是NO_VALUE
public fun <T> MutableStateFlow(value: T): MutableStateFlow<T> = StateFlowImpl(value ?: NULL)
```

#####  emit
> 每次`emit`值就是更新成员变量`value`的值，内部通过`CAS`和`synchronized`来实现线程安全的

```kotlin
override fun tryEmit(value: T): Boolean {
    this.value = value
    return true
}

override suspend fun emit(value: T) {
    this.value = value
}

//state
private val _state = atomic(initialState)

//value的set/get
public override var value: T
    get() = NULL.unbox(_state.value)
    set(value) { updateState(null, value ?: NULL) }
```


```kotlin
private fun updateState(expectedState: Any?, newState: Any): Boolean {
        //...
        
        synchronized(this) {
            val oldState = _state.value
            if (expectedState != null && oldState != expectedState) return false 
            if (oldState == newState) return true
            //更新的是state的value
            _state.value = newState
            //...
        }
        //...
    }

```


#####  collect

```kotlin
override suspend fun collect(collector: FlowCollector<T>) {
    val slot = allocateSlot()
    try {
        if (collector is SubscribedFlowCollector) collector.onSubscription()
        val collectorJob = currentCoroutineContext()[Job]
        var oldState: Any? = null
        
        while (true) {
            
            val newState = _state.value
            
            collectorJob?.ensureActive()
            
            //比对 新老state
            if (oldState == null || oldState != newState) {
               
               //这里的逻辑就是，如果是NULL就emit(null)，否则emit(value) collector.emit(NULL.unbox(newState))
                oldState = newState
            }
            
            //...
        }
    } finally {
        freeSlot(slot)
    }
}

```

#### 第二种: 通过Flow.stateIn方式实现

##### stateIn()

```kotlin
public fun <T> Flow<T>.stateIn(
    scope: CoroutineScope,
    started: SharingStarted,
    initialValue: T
): StateFlow<T> {
    //和shareIn有点像，这里config的replay值固定是1
    val config = configureSharing(1)
    //创建MutableStateFlow，传入默认值
    val state = MutableStateFlow(initialValue)
    //和shareIn实现一样
    val job = scope.launchSharing(config.context, config.upstream, state, started, initialValue)
    //这里创建的是ReadonlyStateFlow，和shareIn没啥区别
    return ReadonlyStateFlow(state, job)
}
```
##### emit&collect
> 和`MutableStateFlow`那套一样的逻辑，区别就是走stateIn逻辑的会先触发`collect`，发送一次默认值，这也解释了为什么`stateIn`逻辑`collect`会收到默认值。

### 思考StateFlow和SharedFlow的使用场景
> 区分要处理的场景是“状态”还是“事件”，状态需要一个默认值，比如UI，事件不需要。

### StateFlow和LiveData(官方原话)
> StateFlow 和 LiveData 具有相似之处。两者都是可观察的数据容器类，并且在应用架构中使用时，两者都遵循相似模式。

> 但请注意，StateFlow 和 LiveData 的行为确实有所不同：

- StateFlow 需要将初始状态传递给构造函数，而 LiveData 不需要。
    
- 当 View 进入 STOPPED 状态时，LiveData.observe() 会自动取消注册使用方，而从 StateFlow 或任何其他数据流收集数据的操作并不会自动停止。如需实现相同的行为，您需要从 Lifecycle.repeatOnLifecycle 块收集数据流。

## ChannelFlow
> 基于`Channel`实现的数据流，本质是Channel


### 什么是Channel？
> `Channel`一般是被用于协程间通信的工具，一个`Channel`由`SendChannel`和`ReceiveChannel`两个部分组成，`SendChannel`用于发送数据，`ReceiveChannel`用于接收数据。`Channel`与阻塞队列有点像，`SendChannel`底层数据结构是一个链表，用于缓存数据。与阻塞队列不同的是，`Channel`适用挂起替代阻塞。

```kotlin
//源码里的一段channel机制伪代码实现

// Create the channel with onUndeliveredElement block that closes a resource
val channel = Channel<Resource>(capacity) { resource -> resource.close() }

// Producer code
val resourceToSend = openResource()
channel.send(resourceToSend)

// Consumer code
val resourceReceived = channel.receive()
try {
    // work with received resource
} finally {
    resourceReceived.close()
}

```

### ChannelFlow简单实现
```kotlin
//举个例子
//与其他flow不同的是，channelFlow调用send发送数据
val flow = channelFlow {
    send("hello channel flow")
}

//收集数据还是collect
flow.collect {
    Log.i("Blues", "receive channel flow: $it")
}


//输出
//receive channel flow: hello channel flow
```

### channelFlow创建

```kotlin
//Builders.kt
public fun <T> channelFlow(@BuilderInference block: suspend ProducerScope<T>.() -> Unit): Flow<T> =
    ChannelFlowBuilder(block)
    
    
private open class ChannelFlowBuilder<T>(
    private val block: suspend ProducerScope<T>.() -> Unit,
    context: CoroutineContext = EmptyCoroutineContext,
    capacity: Int = BUFFERED,
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND
) : ChannelFlow<T>(context, capacity, onBufferOverflow) {
    //执行block代码块
    override suspend fun collectTo(scope: ProducerScope<T>) =
        block(scope)
}
```

### collect

```kotlin
//ChannelFlow.kt
override suspend fun collect(collector: FlowCollector<T>): Unit =
        coroutineScope {
            //先关注produceImpl的实现
            collector.emitAll(produceImpl(this))
        }

//返回的是ReceiveChannel，注意传入的block
//然后里面的操作就是开启协程，执行block代码，执行的就是collectToFun函数
public open fun produceImpl(scope: CoroutineScope): ReceiveChannel<T> =
    scope.produce(context, produceCapacity, onBufferOverflow, start = CoroutineStart.ATOMIC, block = collectToFun)

//Produce.kt
internal fun <E> CoroutineScope.produce(
    context: CoroutineContext = EmptyCoroutineContext,
    capacity: Int = 0,
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    onCompletion: CompletionHandler? = null,
    @BuilderInference block: suspend ProducerScope<E>.() -> Unit
): ReceiveChannel<E> {
    //...
    coroutine.start(start, coroutine, block)
}

 
//调的ChannelFlowBuilder#collectTo()
internal val collectToFun: suspend (ProducerScope<T>) -> Unit
        get() = { collectTo(it) }

//这个block就是channelFlow{}里的block。这个block里的操作就是调用send()发送数据    
override suspend fun collectTo(scope: ProducerScope<T>) = block(scope)

```


```kotlin
//Channels.kt
public suspend fun <T> FlowCollector<T>.emitAll(channel: ReceiveChannel<T>): Unit =
    emitAllImpl(channel, consume = true)
    

private suspend fun <T> FlowCollector<T>.emitAllImpl(channel: ReceiveChannel<T>, consume: Boolean) {

        try {
        while (true) {
            
            //传入的channel是ReceiveChannel，调用其receive
            val result = run { channel.receiveCatching() }
            
            //结束条件就是channel被关闭
            if (result.isClosed) {
                result.exceptionOrNull()?.let { throw it }
                break
            }
            
            //这里就是collect{}里的的block
            emit(result.getOrThrow())
        }
    } catch (e: Throwable) {
        cause = e
        throw e
    } finally {
        //传入的consume是true，最终会取消这个channel
        if (consume) channel.cancelConsumed(cause)
    }
}

```

### send & receive
> 涉及到`Channel`的机制


## 如何取消Flow
> `Flow`没有相关取消的`api`，`flow`是基于协程运行的，所以可以通过取消协程的方式取消`flow`

```kotlin
//example
val flow = flow {
    //发送10次
    repeat(10) {
        //每次延迟100ms
        delay(100)
        emit("hello flow~ $it")
    }
}

val job = launch {
    flow.collect {
        Log.i("Blues", "collect flow:$it")
    }
}
delay(500)
job.cancel()

//这里只输出了4次
//collect flow:hello flow~ 0
//collect flow:hello flow~ 1
//collect flow:hello flow~ 2
//collect flow:hello flow~ 3
```

## 注意事项

- 事例1
```kotlin
val shareFlow = MutableSharedFlow<Int>(2)

for (i in 0..3) {
    shareFlow.emit(i)
}

//collect1和collect2是否都会打印？
lifecycleScope.launch {
    shareFlow.collect {
        Log.i("Blues", "collect1  it:$it")
    }

    shareFlow.collect {
        Log.i("Blues", "collect2  it:$it")
    }
}
```

> lifecycleScope.launch启动的环境是UI线程，SharedFlow#collect是挂起函数，collect内部实现双while(true)循环等待值，会导致后续方法不会被执行。

```kotlin
//collect1和collect2都会打印
val shareFlow = MutableSharedFlow<Int>(2)

for (i in 0..3) {
    shareFlow.emit(i)
}

lifecycleScope.launch {
    shareFlow.collect {
        Log.i("Blues", "share flow by shareIn collect1  it:$it")
    }
}

lifecycleScope.launch {
    shareFlow.collect {
        Log.i("Blues", "share flow by shareIn collect2  it:$it")
    }
}
```

- 事例2

```kotlin
//会打印多少次？
val stateFlow = MutableStateFlow(-1)
lifecycleScope.launch {

    stateFlow.collect {
        Log.i("Blues", "collect:$it")
    }
}

lifecycleScope.launch {
    stateFlow.emit(111)
}

lifecycleScope.launch {
    stateFlow.emit(111)
}

```
> `StateFlow`的防抖机制: 每次`emit`的时候都会判断两个值是否一样，一样就不会更新`state`

- 事例3
```kotlin
///输出的结果是什么？
val shareFlow = MutableSharedFlow<Int>(2)

lifecycleScope.launch {
    for (i in 0..3) {
        shareFlow.emit(i)
    }
}

lifecycleScope.launch {
    shareFlow.collect {
        Log.i("Blues", "share flow by shareIn collect1  it:$it")
    }
}


lifecycleScope.launch {
    for (i in 4..7) {
        shareFlow.emit(i)
    }
}

//答案是会输出2、3、4、5、6、7
```

- 事例4
```kotlin
//把上述第二次的emit换成tryEmit会打印什么结果呢？

//答案是会输出 2、3、4、5
```


1. 调用`emit`发射数据，产生溢流时`tryEmitLocked`返回`false`然后会调用`emitSuspend`发送`Emitter`到`buffer`缓冲区。
2. 调用`tryEmit`，溢流时直接返回`false`不会走`enqueueLocked`逻辑，后续的数据不会入队，所以只有前面2个数据入队了，`collect`时也就只有前2个数据。

```kotlin
private fun tryEmitLocked(value: T): Boolean {

    //emit溢流时会返回false，然后调用emitSuspend，发送一个Emitter入队
    if (bufferSize >= bufferCapacity && minCollectorIndex <= replayIndex) {
        when (onBufferOverflow) {
            BufferOverflow.SUSPEND -> return false 
            BufferOverflow.DROP_LATEST -> return true 
            BufferOverflow.DROP_OLDEST -> {} 
        }
    }
    //不溢流正常放入缓冲区
    enqueueLocked(value)
    bufferSize++ 
    //因为队列的操作是先入队再制空，所以可能会触发丢弃操作，如果replayIndex<head就会更新replayIndex的值，也就是会影响上述的判断可能会进入when判断。
    if (bufferSize > bufferCapacity) dropOldestLocked()
   
    if (replaySize > replay) {
        updateBufferLocked(replayIndex + 1, minCollectorIndex, bufferEndIndex, queueEndIndex)
    }
    return true
}

```

3. 使用`emit`发送数据产生溢流时，，在`collect`的时候`tryTakeValue`会更新缓冲区
```kotlin
private fun tryTakeValue(slot: SharedFlowSlot): Any? {
    var resumes: Array<Continuation<Unit>?> = EMPTY_RESUMES
    val value = synchronized(this) {
        val index = tryPeekLocked(slot)
        if (index < 0) {
            NO_VALUE
        } else {
            val oldIndex = slot.index
            val newValue = getPeekedValueLockedAt(index)
            slot.index = index + 1 
            
            //这里会更新缓冲区buffer
            resumes = updateCollectorIndexLocked(oldIndex)
            newValue
        }
    }
    for (resume in resumes) resume?.resume(Unit)
    return value
}


//在这里实现替换emitter为emitter.value
internal fun updateCollectorIndexLocked(oldIndex: Long): Array<Continuation<Unit>?> {

    //...代码太多省略...
    
    //拿到Emitter
    val emitter = buffer.getBufferAt(curEmitterIndex)
    if (emitter !== NO_VALUE) {
        //强转一下
        emitter as Emitter
        resumes[resumeCount++] = emitter.cont
        
        //重置一下emitter的值为NO_VALUE
        buffer.setBufferAt(curEmitterIndex, NO_VALUE)
        
        //在buffer尾部插入emitter的值
        buffer.setBufferAt(newBufferEndIndex, emitter.value)
        newBufferEndIndex++
        if (resumeCount >= maxResumeCount) break
    }
    
    //省略部分代码...
    
    //这里会将head到newHead的值为null
    updateBufferLocked(newReplayIndex, newMinCollectorIndex, newBufferEndIndex, newQueueEndIndex)
    
    //清楚尾部所有的NO_VALUE项，然后精简queueSize
    cleanupTailLocked()
}

private fun updateBufferLocked(
    newReplayIndex: Long,
    newMinCollectorIndex: Long,
    newBufferEndIndex: Long,
    newQueueEndIndex: Long
) {
    //省略部分代码...
    val newHead = minOf(newMinCollectorIndex, newReplayIndex)
    
    //置空操作
    for (index in head until newHead) buffer!!.setBufferAt(index, null)
}

private fun cleanupTailLocked() {
    //...
    val buffer = buffer!!
    
    //while循环置空buffer尾部的NO_VALUE项
    while (queueSize > 0 && buffer.getBufferAt(head + totalSize - 1) === NO_VALUE) {
        queueSize--
        buffer.setBufferAt(head + totalSize, null)
    }
}
```
