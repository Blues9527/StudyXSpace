[toc]

# Coil-缓存篇

> Coil是一个结合Kotlin语言特性和语法糖编写的图片加载库，Coil的含义就是 Coroutine Image Loader的简写。

## 简单使用

```kotlin
//exmaple 1
val url = "https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png"
val request = ImageRequest.Builder(this).data(url).target { drawable ->
    findViewById<ImageView>(R.id.iv_example).setImageDrawable(drawable)

}.build()
this.imageLoader.enqueue(request)

//example2
findViewById<ImageView>(R.id.iv_example).load(url)
```


- 更快: Coil 在性能上有很多优化，包括内存缓存和磁盘缓存，把缩略图存保存在内存中，循环利用 bitmap，自动暂停和取消图片网络请求等。
- 更轻量级: Coil 只有2000个方法（前提是你的 APP 里面集成了 OkHttp 和 Coroutines），Coil 和 Picasso 的方法数差不多，相比 Glide 和 Fresco 要轻量很多。
- 更容易使用: Coil 的 API 充分利用了 Kotlin 语言的新特性，简化和减少了很多样板代码。
- 更流行: Coil 首选 Kotlin 语言开发并且使用包含 Coroutines, OkHttp, Okio 和 AndroidX Lifecycles 在内最流行的开源库。


## 缓存策略-CachePolicy

```kotlin
//缓存策略主要是限制可读性和可写性，缓存策略适用于内存缓存，磁盘缓存以及网络缓存
enum class CachePolicy(
    val readEnabled: Boolean,
    val writeEnabled: Boolean
) {
    ENABLED(true, true),//可读可写，默认的
    READ_ONLY(true, false),//只读
    WRITE_ONLY(false, true),//只写
    DISABLED(false, false)//都不可
}
```

## 请求数据时的策略-HttpFetcher

- 磁盘缓存和网络缓存都是通过okhttp的cacheControl间接实现的

```kotlin
//表示是否网络可以拉取数据
val networkRead = options.networkCachePolicy.readEnabled
//表示是否可以从磁盘获取数据
val diskRead = options.diskCachePolicy.readEnabled
when {
    //网络不可读，磁盘可读，读磁盘的
    !networkRead && diskRead -> {
        request.cacheControl(CacheControl.FORCE_CACHE)
    }
    //网络可读，磁盘不可读，再判断
    //如果磁盘可写，走网络请求，缓存到disk
    networkRead && !diskRead -> if (options.diskCachePolicy.writeEnabled) {
        request.cacheControl(CacheControl.FORCE_NETWORK)
    } else {
        //磁盘不可写，走网络请求不缓存
        request.cacheControl(CACHE_CONTROL_FORCE_NETWORK_NO_CACHE)
    }
    //网络不可读，磁盘也不可读，会抛出一个504异常
    !networkRead && !diskRead -> {
        request.cacheControl(CACHE_CONTROL_NO_NETWORK_NO_CACHE)
    }
}

```


```kotlin
//EngineInterceptor.intercept()

//如果内存缓存可读，从memoryCacheService获取缓存
val value = if (request.memoryCachePolicy.readEnabled) memoryCacheService[memoryCacheKey] else null


//缓存有效就返回SuccessResult
if (value != null && isCachedValueValid(memoryCacheKey, value, request, size)) {
    return SuccessResult(
        drawable = value.bitmap.toDrawable(context),
        request = request,
        metadata = Metadata(
            memoryCacheKey = memoryCacheKey,
            isSampled = value.isSampled,
            dataSource = DataSource.MEMORY_CACHE,
            isPlaceholderMemoryCacheKeyPresent = chain.cached != null
        )
    )
}

// Fetch, decode, transform, and cache the image on a background dispatcher.
return withContext(request.dispatcher) {

    //标记为无效
    invalidateData(request.data)
    
    //value.count--
    if (value != null) referenceCounter.decrement(value.bitmap)

    // Fetch and decode the image.
    val (drawable, isSampled, dataSource) =
        execute(mappedData, fetcher, request, chain.requestType, size, eventListener)

    // 标记为有效
    validateDrawable(drawable)

    // 写入内存缓存
    val isCached = writeToMemoryCache(request, memoryCacheKey, drawable, isSampled)

    // 返回结果
    SuccessResult(
        drawable = drawable,
        request = request,
        metadata = Metadata(
            memoryCacheKey = memoryCacheKey.takeIf { isCached },
            isSampled = isSampled,
            dataSource = dataSource,
            isPlaceholderMemoryCacheKeyPresent = chain.cached != null
        )
    )
}

private fun writeToMemoryCache(
    request: ImageRequest,
    key: MemoryCache.Key?,
    drawable: Drawable,
    isSampled: Boolean
): Boolean {
    //先判断内存缓存是否可写，不可写直接return false
    if (!request.memoryCachePolicy.writeEnabled) {
        return false
    }

    if (key != null) {
        val bitmap = (drawable as? BitmapDrawable)?.bitmap
        if (bitmap != null) {
            //放入StrongMemoryCache
            strongMemoryCache.set(key, bitmap, isSampled)
            return true
        }
    }
    return false
}

```

> 磁盘缓存只会缓存原尺寸的图

### 技术小结
> Coil实现网络缓存/磁盘缓存主要通过CachePolicy来实现读写的控制，通过OkHttp的request.cacheControl()来控制。根据网络是否可读，磁盘是否可读区分不同的情况采用不同的实现。由于网络缓存不需要写只有读，所以网络的可写权限不需要判断。


## 内存缓存-RealMemoryCache
> 会持有StrongMemoryCache、WeakMemoryCache、BitmapReferenceCounter，相当于一个Manager的角色。

- 默认的BitmapPool大小会根据SDK版本变化，api >= 24 -> 0M   api >= 19 -> 25.6M  else -> 12.8M
- 默认的内存缓存大小是 api >= 24 -> 51.2M   api >= 19 -> 25.6M  else -> 38.4M
- 计算公式是: 可用内存 = BitmapPool大小+内存缓存大小

```
//构造函数
RealMemoryCache(
    val strongMemoryCache: StrongMemoryCache,
    val weakMemoryCache: WeakMemoryCache,
    val referenceCounter: BitmapReferenceCounter,
    val bitmapPool: BitmapPool
)
```

### 核心实现
#### get()

```kotlin
override fun get(key: Key): Bitmap? {
    //先从StrongMemoryCache取，取不到再从WeakMemoryCache取。注意，这里从WeakMemoryCache取出来的是StrongValue，也就是直接持有Bitmap对象的包装类
    val value = strongMemoryCache.get(key) ?: weakMemoryCache.get(key)
    //如果取到了，通过BitmapReferenceCounter标记其无效
    return value?.bitmap?.also { referenceCounter.setValid(it, false) }
}
```

#### set()

```kotlin
override fun set(key: Key, bitmap: Bitmap) {
    //标记无效
    referenceCounter.setValid(bitmap, false)
    //设置进lru缓存
    strongMemoryCache.set(key, bitmap, false)
    //从弱引用缓存移除
    weakMemoryCache.remove(key) 
}
```

#### remove()
```kotlin
override fun remove(key: Key): Boolean {
    // 从lru缓存和弱引用缓存都移除
    val removedStrong = strongMemoryCache.remove(key)
    val removedWeak = weakMemoryCache.remove(key)
    return removedStrong || removedWeak
}
```

#### clear()

```kotlin
override fun clear() {
    //清空lru缓存和弱引用缓存
    strongMemoryCache.clearMemory()
    weakMemoryCache.clearMemory()
}
```

### 技术小结
> 由上可看出，从内存缓存取的话是先从 StrongMemoryCache(也就是LruCache)取，取不到的话就再从WeakMemoryCache取。如果最终取到bitmap了，BitmapReferenceCounter就会标记其为isValid = false


## 弱引用内存缓存-RealWeakMemoryCache 
> 存储的数据结构是 HashMap

```kotlin
internal val cache = hashMapOf<Key, ArrayList<WeakValue>>()

```
### Key
> Key有两种，分别是Simple和Complex

```kotlin

//simple
internal data class Simple(val value: String) : Key()


//complex，不能直接创建实例，因为不能同步去创建
internal data class Complex(
    val base: String,
    val transformations: List<String>,
    val size: Size?,
    val parameters: Map<String, String>
) : Key()
```

### WeakValue

```kotlin
internal class WeakValue(
    val identityHashCode: Int,
    val bitmap: WeakReference<Bitmap>,
    val isSampled: Boolean,
    val size: Int
)

```
### StrongValue

```kotlin
private class StrongValue(
    override val bitmap: Bitmap,
    override val isSampled: Boolean
) : Value
```

### 具体api实现
#### get()

```
@Synchronized
override fun get(key: Key): Value? {
    //取值
    val values = cache[key] ?: return null

    //找到arraylist中第一个不为空的bitmap
    val strongValue = values.firstNotNullIndices { value ->
        //找到后会返回一个使用StrongValue包裹的value对象
        value.bitmap.get()?.let { StrongValue(it, value.isSampled) }
    }

    //检测一下是否需要clean up
    cleanUpIfNecessary()
    
    return strongValue
}
```
#### set()

```
@Synchronized
override fun set(key: Key, bitmap: Bitmap, isSampled: Boolean, size: Int) {

    //尝试获取值，没有就返回 arrayListOf()
    val values = cache.getOrPut(key) { arrayListOf() }

    //按size降序插入到list
    run {
        //计算 hashcode
        val identityHashCode = bitmap.identityHashCode
        //构建一个WeakValue
        val newValue = WeakValue(identityHashCode, WeakReference(bitmap), isSampled, size)
        
        for (index in values.indices) {
            val value = values[index]
            if (size >= value.size) {
            
                //同一个，执行替换
                if (value.identityHashCode == identityHashCode && value.bitmap.get() === bitmap) {
                    values[index] = newValue
                } else {
                    //添加到对应的位置
                    values.add(index, newValue)
                }
                //通过@label跳出run lambda
                return@run
            }
        }
        //+=的含义就是add操作，这里是添加到list尾部
        values += newValue
    }

    //检测一下是否需要clean up
    cleanUpIfNecessary()
}
```

#### remove()

```kotlin
@Synchronized
override fun remove(key: Key): Boolean {
    //逻辑很简单，直接从map中移除掉
    return cache.remove(key) != null
}
```

```kotlin
@Synchronized
override fun remove(bitmap: Bitmap): Boolean {
    val identityHashCode = bitmap.identityHashCode

    //这里的逻辑也很简单，遍历通过hashcode看是否能找到对应的bitmap，找到就移除
    val removed = run {
        cache.values.forEach { values ->
            for (index in values.indices) {
                if (values[index].identityHashCode == identityHashCode) {
                    values.removeAt(index)
                    return@run true
                }
            }
        }
        return@run false
    }

    //检测一下是否需要clean up
    cleanUpIfNecessary()
    return removed
}
```
#### clear()

```kotlin
@Synchronized
override fun clearMemory() {
    //重置操作数
    operationsSinceCleanUp = 0
    //清除map的内容
    cache.clear()
}
```

#### trim()

```kotlin
@Synchronized
override fun trimMemory(level: Int) {
    //判断是否达到clean up条件
    if (level >= TRIM_MEMORY_RUNNING_LOW && level != TRIM_MEMORY_UI_HIDDEN) {
        cleanUp()
    }
}
```

#### cleanup()

```kotlin
private fun cleanUpIfNecessary() {
    //这里的CLEAN_UP_INTERVAL 间隔是10，每次调用都会++
    if (operationsSinceCleanUp++ >= CLEAN_UP_INTERVAL) {
        cleanUp()
    }
}
```


```kotlin
internal fun cleanUp() {
    
    //重置操作数
    operationsSinceCleanUp = 0

    //移除所有被回收的value
    val iterator = cache.values.iterator()
    while (iterator.hasNext()) {
        val list = iterator.next()

        if (list.count() <= 1) {
            // 判断是否是空的情况
            if (list.firstOrNull()?.bitmap?.get() == null) {
                iterator.remove()
            }
        } else {
            //区分不同的SDK版本执行对应的操作，移除list中bitmap为空的value
            if (SDK_INT >= 24) {
                list.removeIf { it.bitmap.get() == null }
            } else {
                list.removeIfIndices { it.bitmap.get() == null }
            }

            //上面移除完后list是空的，则把list也移除了
            if (list.isEmpty()) {
                iterator.remove()
            }
        }
    }
}
```

## 强引用内存缓存-RealStrongMemoryCache

```kotlin
//构造函数
RealStrongMemoryCache(
    private val weakMemoryCache: WeakMemoryCache,//弱引用缓存
    private val referenceCounter: BitmapReferenceCounter,//引用计数器
    maxSize: Int,
    private val logger: Logger?
)
```
> 存储的数据数据结构-LruCache

```kotlin
private val cache = object : LruCache<Key, InternalValue>(maxSize) {
    override fun entryRemoved(
        evicted: Boolean,
        key: Key,
        oldValue: InternalValue,
        newValue: InternalValue?
    ) {
        //entry被移除的时候，调用 引用计数器decrement一下
        val isPooled = referenceCounter.decrement(oldValue.bitmap)
        if (!isPooled) {
            //如果bitmap不只添加到BitmapPool，会将其添加到WeakMemoryCache
            weakMemoryCache.set(key, oldValue.bitmap, oldValue.isSampled, oldValue.size)
        }
    }

    override fun sizeOf(key: Key, value: InternalValue) = value.size
}
```
### InternalValue

```kotlin
private class InternalValue(
    override val bitmap: Bitmap,
    override val isSampled: Boolean,
    val size: Int
) : Value
```

### 其他关键的api

#### get()

```kotlin
@Synchronized
override fun get(key: Key): Value? = cache.get(key)//直接从Lru中取
```

#### set()
```kotlin
@Synchronized
override fun set(key: Key, bitmap: Bitmap, isSampled: Boolean) {
    
    //这里为什么需要计算bitmap大小呢？因为bitmap太大的话，是不会尝试存储的。因为这样做会导致缓存被清除。
    
    //计算bitmap占用的字节大小 width * height * config.bytesPerPixel
    val size = bitmap.allocationByteCountCompat
    if (size > maxSize) {
        //如果bitmap过大，就从缓存移除掉
        val previous = cache.remove(key)
        if (previous == null) {
            // 如果previous不为空的话，会在LruCache.entryRemoved添加进weakMemoryCache
            weakMemoryCache.set(key, bitmap, isSampled, size)
        }
        return
    }

    //调用引用计数器increment
    referenceCounter.increment(bitmap)
    //放入Lru缓存
    cache.put(key, InternalValue(bitmap, isSampled, size))
}
```

#### remove()

```kotlin
@Synchronized
override fun remove(key: Key): Boolean {
    return cache.remove(key) != null
}
```

#### clear()

```kotlin
@Synchronized
override fun clearMemory() {
    //清空lru
    cache.trimToSize(-1)
}
```
 
#### trim()

```kotlin
@Synchronized
override fun trimMemory(level: Int) {
    //根据不同的情况对lrc进行裁剪
    if (level >= TRIM_MEMORY_BACKGROUND) {
        clearMemory()
    } else if (level in TRIM_MEMORY_RUNNING_LOW until TRIM_MEMORY_UI_HIDDEN) {
        cache.trimToSize(size / 2)
    }
}

```

## Bitmap引用计数器-RealBitmapReferenceCounter

```kotlin
//构造函数
RealBitmapReferenceCounter(
    private val weakMemoryCache: WeakMemoryCache,
    private val bitmapPool: BitmapPool,
    private val logger: Logger?
)


internal val values = SparseArrayCompat<Value>()
internal var operationsSinceCleanUp = 0
```

### Value

```kotlin
internal class Value(
    val bitmap: WeakReference<Bitmap>,
    var count: Int,
    var isValid: Boolean
)
```


### 关键方法

#### increment()

```kotlin
@Synchronized
override fun increment(bitmap: Bitmap) {

    //计算hash code
    val key = bitmap.identityHashCode
    
    //拿到or生成一个Value
    val value = getValue(key, bitmap)
    
    //记数
    value.count++
    
    //跟WeakMemoryCache的实现一致
    cleanUpIfNecessary()
}
```

#### decrement()

```kotlin
@Synchronized
override fun decrement(bitmap: Bitmap): Boolean {
    val key = bitmap.identityHashCode
    val value = getValueOrNull(key, bitmap) ?: run {
        return false
    }
    
    //记数
    value.count--

    // If the bitmap is valid and its count reaches 0, remove it
    // from the WeakMemoryCache and add it to the BitmapPool.
    
    //如果bitmap是标记为有效的，但是记数达到了0，会从WeakMemoryCache移除并加入到BitmapPool
    val removed = value.count <= 0 && value.isValid
    if (removed) {
        values.remove(key)
        weakMemoryCache.remove(bitmap)
        //使用异步的方式，在下一帧添加到bitmapPool
        MAIN_HANDLER.post { bitmapPool.put(bitmap) }
    }

    cleanUpIfNecessary()
    return removed
}

```

#### setValid()

```kotlin
@Synchronized
override fun setValid(bitmap: Bitmap, isValid: Boolean) {
    val key = bitmap.identityHashCode
    if (isValid) {
        //尝试获取Value
        val value = getValueOrNull(key, bitmap)
        if (value == null) {
            //实例化一个Value并放入values里
            values[key] = Value(WeakReference(bitmap), 0, true)
        }
    } else {
        //拿到对应的value，并标记为isValid = false
        val value = getValue(key, bitmap)
        value.isValid = false
    }
    cleanUpIfNecessary()
}
```

#### cleanup()

```kotlin
internal fun cleanUp() {
    //使用arrayList来记录要被移除的index
    val toRemove = arrayListOf<Int>()
    for (index in 0 until values.size) {
        val value = values.valueAt(index)
        if (value.bitmap.get() == null) {
            toRemove += index
        }
    }
    //执行removeAt操作
    toRemove.forEachIndices(values::removeAt)
}
```

### get()衍生操作

```kotlin
    private fun getValue(key: Int, bitmap: Bitmap): Value {
        //尝试获取，获取不到 实力话一个Value
        var value = getValueOrNull(key, bitmap)
        if (value == null) {
            value = Value(WeakReference(bitmap), 0, false)
            values[key] = value
        }
        return value
    }

    private fun getValueOrNull(key: Int, bitmap: Bitmap): Value? {
        return values[key]?.takeIf { it.bitmap.get() === bitmap }
    }
```


## Coil扩展
- 支持compose加载图片
- 支持gif加载
- 支持svg加载
- 支持video帧加载


## 与Glide对比
- Glide使用EngineResource的acquired变量来记录bitmap的使用情况，Coil的话使用RealBitmapReferenceCounter来实现，
- Coil的RealBitmapPool就是参考Glide的LruBitmapPool实现的
- Coil只会缓存原图，而Glide更强大会缓存各种图
- Glide取内存缓存的逻辑是先从弱引用缓存取，取不到再从Lru缓存取。而Coil的话会先从StrongMemoryCache取(也就是Lru缓存)，取不到再从WeakMemoryCache缓存取(也就是弱引用缓存)。
- Coil默认使用Okhttp请求，Glide默认HttpUrlConnection，可支持Okhttp扩展
 
