### 源码阅读

#### SparseArray的前生今世
SparseArray位于 android.util包下，实现了Cloneable的接口。

#### 属性

```
    //删除标识
    private static final Object DELETED = new Object();
    
    //是否需要进行垃圾回收
    private boolean mGarbage = false;

    //使用int数组存储key值
    private int[] mKeys;
    //使用object数组存储value值
    private Object[] mValues;
    //数组大小
    private int mSize;
```


#### 构造函数

```
构造一：
public SparseArray() {
        //调用此构造函数默认容量为10
        this(10);
    }
    
构造二：
public SparseArray(int initialCapacity) {
        if (initialCapacity == 0) {
            //如果传入的初始容量为0，则实例化一个int的空数组和object的空数组
            mKeys = EmptyArray.INT;
            mValues = EmptyArray.OBJECT;
        } else {
            mValues = ArrayUtils.newUnpaddedObjectArray(initialCapacity);
            mKeys = new int[mValues.length];
        }
        //数组初始大小为0
        mSize = 0;
    }
```

#### 常用操作

##### 1.get操作
```
public E get(int key) {
        //调用此方法，传入默认value为空，即可能为null
        return get(key, null);
    }
    
public E get(int key, E valueIfKeyNotFound) {
        //因为keys素组是有序的，所以通过二分查找去找到传入的key
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        //如果返回的key下标为负数或者对应的value被标记为删除
        if (i < 0 || mValues[i] == DELETED) {
            //返回默认值
            return valueIfKeyNotFound;
        } else {
            //否则返回对应的值
            return (E) mValues[i];
        }
    }
```
##### 2.delete操作

```
public void delete(int key) {
        //二分查找
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        //如果对应key的下标大于0才执行delete操作
        if (i >= 0) {
            if (mValues[i] != DELETED) {
                //将对应的value标记为DELETED
                mValues[i] = DELETED;
                //复制为需要进行垃圾回收
                mGarbage = true;
            }
        }
    }
```

##### 3.remove操作

```
public void remove(int key) {
        //调用delete方法
        delete(key);
    }
    
//直接通过下标进行删除，方法同delete
public void removeAt(int index) {
        if (mValues[index] != DELETED) {
            mValues[index] = DELETED;
            mGarbage = true;
        }
    }
    
//移除指定key的值并返回旧的值
public E removeReturnOld(int key) {
        //二分查找
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i >= 0) {
            if (mValues[i] != DELETED) {
                //获取旧值
                final E old = (E) mValues[i];
                mValues[i] = DELETED;
                mGarbage = true;
                return old;
            }
        }
        return null;
    }
    
//移除指定范围的值
public void removeAtRange(int index, int size) {
        //计算结束的下标
        final int end = Math.min(mSize, index + size);
        for (int i = index; i < end; i++) {
            //调用removeAt方法，通过传入下标进行移除
            removeAt(i);
        }
    }
```

##### 4.put操作

```
//插值操作
public void put(int key, E value) {
        //二分查找
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i >= 0) {
            //如果已存在对应的key，则直接替换value
            mValues[i] = value;
        } else {
            //整型取~的运算过程是  -(i+1)即 +1取负数
            //这里i已经是<0的了，所以进行~操作后肯定是正数
            i = ~i;

            //如果i小于数组的大小，且对应下标的value值是被标记为DELETED了的，直接插入key&value，return
            if (i < mSize && mValues[i] == DELETED) {
                mKeys[i] = key;
                mValues[i] = value;
                return;
            }

            //上述条件只有进行过delete或remove相关操作才成立，所以一般会进入此判断
            //如果需要进行垃圾回收，且数组的大小大于key数组的长度，则进行gc操作
            if (mGarbage && mSize >= mKeys.length) {
                gc();

                // Search again because indices may have changed.
                //上述的意思即是垃圾回收可能会导致下标发生变化，所以gc后需要重新通过二分查找去获取下标
                i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
            }

            //插值操作，GrowingArrayUtils的insert方法支持动态扩容，即可以根据需要去自动扩容
            mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
            mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);
            //插入后size++
            mSize++;
        }
    }
```

##### 5.clear操作

```
//移除所有key-value的映射，即移除所有value值
public void clear() {
        int n = mSize;
        Object[] values = mValues;

        //遍历置空操作
        for (int i = 0; i < n; i++) {
            values[i] = null;
        }

        //移除后size赋值为0
        mSize = 0;
        //因为移除的是映射，所以不需要垃圾回收
        mGarbage = false;
    }
```

##### 6.append操作

```
public void append(int key, E value) {
        //如果数组大小不等于0，且插入的key小于等于key素组的最后一个下标
        if (mSize != 0 && key <= mKeys[mSize - 1]) {
            //调用put插值操作，return
            put(key, value);
            return;
        }

        //判断是否需要进行gc操作
        if (mGarbage && mSize >= mKeys.length) {
            gc();
        }

        //插值操作
        mKeys = GrowingArrayUtils.append(mKeys, mSize, key);
        mValues = GrowingArrayUtils.append(mValues, mSize, value);
        mSize++;
    }
```

##### 7.其他操作

```
//查询key值
public int keyAt(int index) {
        //先判断是否需要gc操作，因为涉及下标相关的问题
        if (mGarbage) {
            gc();
        }

        return mKeys[index];
    }
    
//查询value值
public E valueAt(int index) {
        //同先进行gc操作
        if (mGarbage) {
            gc();
        }

        return (E) mValues[index];
    }

//更新（插值）操作    
public void setValueAt(int index, E value) {
        //先gc判断，再赋值
        if (mGarbage) {
            gc();
        }

        mValues[index] = value;
    }

//查找key值的下标    
public int indexOfKey(int key) {
        if (mGarbage) {
            gc();
        }

        return ContainerHelpers.binarySearch(mKeys, mSize, key);
    }

//查找value值的下标
public int indexOfValue(E value) {
        if (mGarbage) {
            gc();
        }

        for (int i = 0; i < mSize; i++) {
            if (mValues[i] == value) {
                return i;
            }
        }

        return -1;
    }

//通过value值去获取value的下标    
public int indexOfValueByValue(E value) {
        if (mGarbage) {
            gc();
        }

        for (int i = 0; i < mSize; i++) {
            //传入的value值为空的情况
            if (value == null) {
                //去查找value数组中首个值为空的下标
                if (mValues[i] == null) {
                    return i;
                }
            } else {
                if (value.equals(mValues[i])) {
                    return i;
                }
            }
        }
        //不存在返回-1
        return -1;
    }
```
##### 8.gc操作

```
//gc操作，内部实现
private void gc() {
        //记录回收前的size
        int n = mSize;
        
        int o = 0;
        //存放回收前的keys
        int[] keys = mKeys;
        //存放回收前的values
        Object[] values = mValues;

        //做循环，把剩余的entry重新排序
        for (int i = 0; i < n; i++) {
            Object val = values[i];

            if (val != DELETED) {
                if (i != o) {
                    keys[o] = keys[i];
                    values[o] = val;
                    values[i] = null;
                }
                o++;
            }
        }
        //垃圾回收结束，将mGarbage赋值为false
        mGarbage = false;
        //重新赋值size
        mSize = o;
    }
    

```

##### 9.涉及的二分查找操作

```
static int binarySearch(int[] array, int size, int value) {
        int lo = 0;
        int hi = size - 1;

        while (lo <= hi) {
            final int mid = (lo + hi) >>> 1;
            final int midVal = array[mid];

            if (midVal < value) {
                lo = mid + 1;
            } else if (midVal > value) {
                hi = mid - 1;
            } else {
                return mid;  // value found
            }
        }
        return ~lo;  // value not present
    }
```
##### 10.GrowingArrayUtils插值相关

```
public static int[] insert(int[] array, int currentSize, int index, int element) {
        assert currentSize <= array.length;

        if (currentSize + 1 <= array.length) {
            System.arraycopy(array, index, array, index + 1, currentSize - index);
            array[index] = element;
            return array;
        } else {
            int[] newArray = new int[growSize(currentSize)];
            System.arraycopy(array, 0, newArray, 0, index);
            newArray[index] = element;
            System.arraycopy(array, index, newArray, index + 1, array.length - index);
            return newArray;
        }
    }
    
public static int growSize(int currentSize) {
        //扩容，小于等于4的返回8，大于4的直接翻倍
        return currentSize <= 4 ? 8 : currentSize * 2;
    }    
```



### 参考文献
[[原创]面试还在问 SparseArray？记住 3 句话让你临时把佛脚抱好！](https://juejin.im/post/5da1481e6fb9a04de96e8b72)