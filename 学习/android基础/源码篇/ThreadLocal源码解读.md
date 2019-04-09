# 5.ThreadLocal
#### 什么是ThreadLocal？

> ThreadLocal是一个线程内部数据存储类，通过他可以在指定的线程中存储数据。存储后，只能在指定的线程中获取到存储的数据，对其他线程来说无法获取到数据。

#### ThreadLocal工作原理
- ThreadLocal对象通过Thread.currentThread()方法去获取当前线程对象；
- 当前Thread获取对象内部持有的ThreadLocalMap容器；
- 从ThreadLocalMap容器中使用ThreadLocal对象作为key，操作当前Thread中的变量副本。

#### ThreadLocal导致内存泄漏的原因及解决办法

> ThreadLocal的内存内泄漏的真正原因并不是因为ThreadLocalMap的key使用了弱引用，而是因为ThreadLocalMap和线程Thread的生命周期一样长，没有手动删除Map的中的key才会导致内存泄漏。所以解决ThreadLocal的内存泄漏问题就要每次使用完ThreadLocal，都要记得调用它的remove()方法来清除。

#### Thread Local源码解读

> ThreadLocal里有两个内部类，SuppliedThreadLocal 和 ThreadLocalMap。其中，SuppliedThreadLocal是ThreadLocal的一个扩展类，主要是JDK1.8用来扩展Lambda表达式的支持；ThreadLocalMap是ThreadLocal的静态内部类，也是ThreadLocal实际用于存储的类。ThreadLocalMap还有一个静态内部类Entry，以ThreadLocal对象为key，变量值为value。

## ThreadLocalMap

> 内部Entry类继承自WeakReference，以ThreadLocal对象作为key去存储变量。

> ThreadLocalMap的初始容量是16

> 调用resize()方法进行扩容，每次扩容新的容量是旧容量的2倍。

> 调用rehash()方法解决hash冲突。

## ThreadLocal里的一些重要方法

1. set()

> 调用set()方法去存储值。首先获取当前Thread对象，然后去获取ThreadLocalMap对象，如果ThreadLocalMap对象不为空则调用ThreadLoacalMap对象的set()方法去存储值，key为当前ThreadLocal对象。如果ThreadLocalMap对象为空，则调用createMap()方法去实例化一个ThreadLocalMap对象，key为当前ThreadLocal对象。

```
 Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
```

2. get()

> 调用get()方法去获取存储的值。首先去获取当前Thread的ThreadLocalMap对象，判断是否为空，如果不为空则继续去获取当前ThreadLocalMap的Entry对象，如果Entry不为空则继续获取entry的value值。否则会调用setInitialValue()方法，而该方法里会继续调用initialValue()，此方法默认返回null。

```
Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
```

3. remove()

> 使用完ThreadLocal后需要调用remove()方法进行清楚，否则会造成内存泄漏的问题。

```
  ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
```

4. withInitial()

> 静态方法，返回一个SuppliedThreadLocal扩展类的实例。
