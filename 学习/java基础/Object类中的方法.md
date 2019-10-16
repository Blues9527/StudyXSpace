# Object Api

### 为什么会写这样一个奇怪的md
> 因为在某一次面试的笔试中确实有考到这个题。平时我们对Object这个类绝对不陌生，但是突然要求写出Object中有哪些方法？一下子就有点懵，所以去看了一下Object类的源码，总结了一下Object类中的api


方法名|参数列表|返回类型|异常|注解
---|---|---|---|---
getClass() || Class<?> | 
hashCode() || int|
identityHashCode() |Object| int||@static
identityHashCodeNative() |Object| int||@native @static
equals() |Object| boolean||
clone() || Object|CloneNotSupportedException|
internalClone() || Object||@native
toString() || String||
notify() || void||@native
notifyAll() || void||@native
wait() |long| void|InterruptedException|
wait() |long,int| void|InterruptedException|@native
wait() || void|InterruptedException|@native
finalize() || void|Throwable|






