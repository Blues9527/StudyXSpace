# LruCache

### 什么是LruCache
> LRU（Least Recently Used） 翻译过来就是 最近最少使用的 意思；


```
定义变量

//LruCache低层是以LinkedHashMap作为数据存储
private final LinkedHashMap<K, V> map;
//LinkedHashMap的大小
private int size;
//缓存的最大值
private int maxSize;
//插入值的次数
private int putCount;
//创建value值的次数
private int createCount;
//回收次数
private int evictionCount;
//插值时命中的次数
private int hitCount;
//插值时丢失的次数
private int missCount;
```


```
构造函数

/**
*   @params maxSize 传入一个缓存最大值
*/
public LruCache(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    } else {
        this.maxSize = maxSize;
        //实例化一个LinkedHashMap，初始容量为0，扩容因子为0.75f， 
        // false： 基于插入顺序     true：  基于访问顺序 
        this.map = new LinkedHashMap(0, 0.75F, true);
    }
}
```

```
重设缓存的大小

public void resize(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    } else {
        synchronized(this) {
            this.maxSize = maxSize;
        }
        //调用trimToSize去调整大小
        trimToSize(maxSize);
    }
}
```

```
调整缓存大小

private void trimToSize(int maxSize) {

    //死循环，跳出条件为 size <= maxSize或者toEvict == null
    while (true) {
        K key;
        V value;
        synchronized (this) {
            //异常条件，抛出异常
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }
            //跳出条件
            if (size <= maxSize) {
                break;
            }
            
            //目的就是取出LinkedHashMap里最后一个项（Lrc）
            Map.Entry<K, V> toEvict = null;
            for (Map.Entry<K, V> entry : map.entrySet()) {
                toEvict = entry;
            }
            //如果最后一项为空，则跳出循环
            if (toEvict == null) {
                break;
            }
            //不为空，则拿到对应的key&value
            key = toEvict.getKey();
            value = toEvict.getValue();
            
            //根据key值调用remove方法移除掉
            map.remove(key);
            
            //然后减去对应的size得到新的size
            size -= safeSizeOf(key, value);
            
            //回收次数++
            evictionCount++;
        }
        entryRemoved(true, key, value, null);
    }
}

//内部实现是调用sizeOf方法，多加一个判断
private int safeSizeOf(K key, V value) {
    int result = sizeOf(key, value);
    if (result < 0) {
        throw new IllegalStateException("Negative size: " + key + "=" + value);
    }
    return result;
}

//entry大小，默认返回1
protected int sizeOf(K key, V value) {
    return 1;
}

//entryRemoved是一个空实现的方法
protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {}
```

```
插值方法，把值插入LinkedHashMap的头部

public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }
    
    V previous;
    synchronized (this) {
        //插值次数++
        putCount++;
        //插值size要加上entry的大小
        size += safeSizeOf(key, value);
        //将值put进LinkedHashMap
        previous = map.put(key, value);
        
        //这里注意一下，HashMap的put方法，如果此前有相同的key值的，则返回此前key值对应的value值，否则就返回空。
        //所以这里判断的是：此前是否有对应的key-value值，不为空则已存在对应的key-value值
        if (previous != null) {
            //可以理解为没有插入新值，LinkedHashMap的大小没有改变，所以要减去此前加上的entry的大小
            size -= safeSizeOf(key, previous);
        }
    }
    
    
    //key-value值已存在，调用entryRemoved（空实现），需要覆写此方法实现逻辑
    if (previous != null) {
        entryRemoved(false, key, previous, value);
    }
    //调整缓存大小
    trimToSize(maxSize);
    return previous;
}
```

```
取值方法

public final V get(K key) {

    //判断传进来的key值是否为空，为空则抛出空指针异常
    if (key == null) {
        throw new NullPointerException("key == null");
    }
    
    V mapValue;
    synchronized (this) {
        //取值
        mapValue = map.get(key);
        if (mapValue != null) {
            //取出来的值不为空，则有值，命中次数++
            hitCount++;
            //将取出来的值返回出去
            return mapValue;
        }
        //否则 丢失次数++
        missCount++;
    } 
    
    //如果取出来的值为空的话，尝试去create一个值
    //要注意的是，create方法默认是返回空的，需要覆写此方法令其不返回空
    V createdValue = create(key);
    //如果create出来的值还是空的话，则返回null
    if (createdValue == null) {
        return null;
    }
    
//这里的理解是，在多线程下，执行完第一个synchronized代码块，取出来的值是null，然后覆写了create方法并返回一个value
//此时，由于多线程的原因，其他线程下对map进行了插值操作，此时对应key是有值的，所以存在put方法时返回的mapValue不为空。
//既然对应key有值了，所以又撤销重新插值的操作
    synchronized (this) {
        //create出来的值不为空，则 创建次数++
        createCount++;
        //重新插值
        mapValue = map.put(key, createdValue);
        //存在冲突
        if (mapValue != null) {
            //值已存在，撤销上面的插值，mapValue为此前存在的值
            map.put(key, mapValue);
        } else {
            //值不存在，插值成功，LinkedHashMap的size+
            size += safeSizeOf(key, createdValue);
        }
    }
    
    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        //修整cache的大小
        trimToSize(maxSize);
        //返回createVale
        return createdValue;
    }
}

//create方法默认返回null
protected V create(K key) {
    return null;
```


```
如果缓存项已存在，则移除掉

public final V remove(K key) {
    //key为空，抛出空指针异常
    if (key == null) {
        throw new NullPointerException("key == null");
    }
    
    V previous;
    synchronized (this) {
        //remove方法会返回一个value
        previous = map.remove(key);
        if (previous != null) {
            //value不为空，移除成功后 减去缓存项的大小
            size -= safeSizeOf(key, previous);
        }
    }
    
    
    if (previous != null) {
        entryRemoved(false, key, previous, null);
    }
    return previous;
}
```

```
清除缓存
public final void evictAll() {
    //调用trimToSize方法传入-1
    trimToSize(-1); // -1 will evict 0-sized elements
}
```




