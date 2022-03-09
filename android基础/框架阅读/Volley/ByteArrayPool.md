# ByteArrayPool类
> byte[]缓冲池，采用LRU策略去清楚缓冲池中无用的byte[]

```
//记录最后使用的byte[]
private final List<byte[]> mBuffersByLastUse = new ArrayList<>();

//实际存储byte[]的数组列表
private final List<byte[]> mBuffersBySize = new ArrayList<>(64);

//记录当前缓冲池的大小
 private int mCurrentSize = 0;
 
 //byte[]池中最大缓冲区的大小，在构造函数中传入
 private final int mSizeLimit;

//传入限定的长度，遍历缓冲池中的byte[]，判断byte[]的大小是否大于传入的len，如果大于传入的len则移除掉，最后返回一个len长度的byte[]
public synchronized byte[] getBuf(int len) {
        for (int i = 0; i < mBuffersBySize.size(); i++) {
            byte[] buf = mBuffersBySize.get(i);
            if (buf.length >= len) {
                mCurrentSize -= buf.length;
                mBuffersBySize.remove(i);
                mBuffersByLastUse.remove(buf);
                return buf;
            }
        }
        return new byte[len];
    }
    
//首先判断传进来的buf是否符合要求，不符合要求则直接return，将使用过的buf放入待删缓冲区mBuffersByLastUse，接着在mBuffersBySize中使用2分查找来确定buf的位置，然后再将buf插入pos位置，最后调用trim()方法去去除多余的byte[]
public synchronized void returnBuf(byte[] buf) {
        if (buf == null || buf.length > mSizeLimit) {
            return;
        }
        mBuffersByLastUse.add(buf);
        int pos = Collections.binarySearch(mBuffersBySize, buf, BUF_COMPARATOR);
        if (pos < 0) {
            pos = -pos - 1;
        }
        mBuffersBySize.add(pos, buf);
        mCurrentSize += buf.length;
        trim();
    }
  
//从池中移除多余的buffers直到给定的大小  
private synchronized void trim() {
        while (mCurrentSize > mSizeLimit) {
            byte[] buf = mBuffersByLastUse.remove(0);
            mBuffersBySize.remove(buf);
            mCurrentSize -= buf.length;
        }
    }
```