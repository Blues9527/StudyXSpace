# Object Api

### 为什么会写这样一个奇怪的md
> 因为在某一次面试的笔试中确实有考到这个题。平时我们对Object这个类绝对不陌生，但是突然要求写出Object中有哪些方法？一下子就有点懵，所以去看了一下Object类的源码，总结了一下Object类中的api

```
getClass() //Class<?>

hashCode() //int

@static
identityHashCode(Object obj) //int

@native @static
identityHashCodeNative(Object obj) //int

equals(Object obj) //boolean

clone() throws CloneNotSupportedException // Object

@native
internalClone() //Object

toString() //String

@native
notify() //void

@native
notifyAll()  //void

wait(long millis) throws InterruptedException //void

@native
wait(long millis, int nanos) throws InterruptedException //void

@native
wait() throws InterruptedException  //void

finalize() throws Throwable //void
```







