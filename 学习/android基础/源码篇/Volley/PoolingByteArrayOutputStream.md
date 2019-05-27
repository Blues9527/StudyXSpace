
```
public class PoolingByteArrayOutputStream extends ByteArrayOutputStream {
    //定义一个初始化byte[]容量为256，若要自定义size则要求必须大于256
    private static final int DEFAULT_SIZE = 256;

    private final ByteArrayPool mPool;

    //调用下方构造方法，并传入默认size，即获取到的buf的大小一定是256
    public PoolingByteArrayOutputStream(ByteArrayPool pool) {
        this(pool, DEFAULT_SIZE);
    }

    //通过传进来的size与默认size进行比较取较大的那个作为获取到byte[]的size
    public PoolingByteArrayOutputStream(ByteArrayPool pool, int size) {
        mPool = pool;
        buf = mPool.getBuf(Math.max(size, DEFAULT_SIZE));
    }

    //调用ByteArrayPool的returnBuf()方法里的trim()方法，将其移除掉并置空
    @Override
    public void close() throws IOException {
        mPool.returnBuf(buf);
        buf = null;
        super.close();
    }

    @Override
    public void finalize() {
        mPool.returnBuf(buf);
    }

    //返回新的buf的size 为 (count + i) * 2，通过System.arraycopy进行copy
    @SuppressWarnings("UnsafeFinalization")
    private void expand(int i) {
        if (count + i <= buf.length) {
            return;
        }
        byte[] newbuf = mPool.getBuf((count + i) * 2);
        System.arraycopy(buf, 0, newbuf, 0, count);
        mPool.returnBuf(buf);
        buf = newbuf;
    }

    @Override
    public synchronized void write(byte[] buffer, int offset, int len) {
        expand(len);
        super.write(buffer, offset, len);
    }

    @Override
    public synchronized void write(int oneByte) {
        expand(1);
        super.write(oneByte);
    }
}
```
