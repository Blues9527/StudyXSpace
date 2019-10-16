#### HashMap
> 初始容量时16，扩容因子是0.75f。调用resize()方法进行扩容，容量是2的幂次方。

#### HashTable
> 默认容量时11，扩容因子是0.75f，调用rehash()方法进行扩容，新容量时 2*初始容量+1。

#### ConCurrentHashMap



#### 解决hash冲突的方法
1. rehash法
2. 开放定址法
3. 链地址法

#### 总结
名称 | 初始容量|扩容因子|扩容方法|新容量|是否线程安全
---|---|---|---|---|---
HashMap | 16| 0.75f| resize()| 2*旧容量| 否
HashTable | 11| 0.75f| rehash()| 2*旧容量 +1| 否
ConCurrentHashMap | 16| 自定| transfer()| | 是

