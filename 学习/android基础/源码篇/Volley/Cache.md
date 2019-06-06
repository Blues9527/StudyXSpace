Cache类是一个接口类，里面还有一个Entry内部类。

**Cache**的一些方法
```
//从缓存中取出一个项
Entry get(String key);

//将一个项放入缓存
void put(String key, Entry entry);

//初始化
void initialize();

//根据释放某个缓存
void invalidate(String key, boolean fullExpire);

//从缓存中移除一个缓存项
void remove(String key);

//清空缓存
void clear();
```

**Entry**类
> 用于存放缓存的数据和元数据的项


```
//cache返回的数据
public byte[] data;

//附着于缓存的tag
public String etag;

//服务器上报的response的日期
public long serverDate;

//最近一次修改的日期
public long lastModified;

//记录TTL
public long ttl;

//记录软TTL
 public long softTtl;
 
 //
 public Map<String, String> responseHeaders = Collections.emptyMap();
 
 //可能为null，取决于具体的实现
 public List<Header> allResponseHeaders;
 
 //是否过期
 public boolean isExpired() {
            return this.ttl < System.currentTimeMillis();
        }

//  是否需要刷新     
public boolean refreshNeeded() {
            return this.softTtl < System.currentTimeMillis();
        }
```

**NoCache**类是Cache的一个实现类，构建一个空的缓存

```

    //空实现
    @Override
    public void clear() {}

    //返回null
    @Override
    public Entry get(String key) {
        return null;
    }

    //空实现
    @Override
    public void put(String key, Entry entry) {}

    //空实现
    @Override
    public void invalidate(String key, boolean fullExpire) {}

    //空实现
    @Override
    public void remove(String key) {}

    //空实现
    @Override
    public void initialize() {}
```

**DiskBasedCache**类是另外一个Cache的实现类


```
//用于存储缓存头的，使用的是链表式的hashmap，初始容量时16，加载因子是0.75f
private final Map<String, CacheHeader> mEntries = new LinkedHashMap<>(16, .75f, true);

//缓存占用的总字节空间
private long mTotalSize = 0;

//缓存根目录
private final File mRootDirectory;

//缓存最大值
private final int mMaxCacheSizeInBytes;

//默认缓存大小为5m
private static final int DEFAULT_DISK_USAGE_BYTES = 5 * 1024 * 1024;

```


```
public DiskBasedCache(File rootDirectory, int maxCacheSizeInBytes) {
        mRootDirectory = rootDirectory;
        mMaxCacheSizeInBytes = maxCacheSizeInBytes;
    }
    
//传入默认的缓存大小为5M
public DiskBasedCache(File rootDirectory) {
        this(rootDirectory, DEFAULT_DISK_USAGE_BYTES);
    }
```

```
//清楚所有的缓存，删除硬盘上所有与的缓存文件
@Override
    public synchronized void clear() {
        File[] files = mRootDirectory.listFiles();
        if (files != null) {
            for (File file : files) {
                file.delete();
            }
        }
        mEntries.clear();
        mTotalSize = 0;
        VolleyLog.d("Cache cleared.");
    }
```


```
@Override
    public synchronized Entry get(String key) {
        CacheHeader entry = mEntries.get(key);
       //如果entry不存在，则return null
        if (entry == null) {
            return null;
        }
        
        //根据传进来的key创建一个文件
        File file = getFileForKey(key);
        try {
            CountingInputStream cis =
                    new CountingInputStream(
                            new BufferedInputStream(createInputStream(file)), file.length());
            try {
                CacheHeader entryOnDisk = CacheHeader.readHeader(cis);
                if (!TextUtils.equals(key, entryOnDisk.key)) {
                   ...
                    removeEntry(key);
                    return null;
                }
                byte[] data = streamToBytes(cis, cis.bytesRemaining());
                return entry.toCacheEntry(data);
            } finally {
                cis.close();
            }
        } catch (IOException e) {
          ...
            remove(key);
            return null;
        }
    }

   
public File getFileForKey(String key) {
        //File(根目录，根目录下的子目录)
        return new File(mRootDirectory, getFilenameForKey(key));
    }
 
//通过传进来的key去创建一个虚假的独一无二的文件名   
private String getFilenameForKey(String key) {
        int firstHalfLength = key.length() / 2;
        String localFilename = String.valueOf(key.substring(0, firstHalfLength).hashCode());
        localFilename += String.valueOf(key.substring(firstHalfLength).hashCode());
        return localFilename;
    }
```


```
 @Override
    public synchronized void initialize() {
        if (!mRootDirectory.exists()) {
            if (!mRootDirectory.mkdirs()) {
            //文件无法被创建，打日志
              ...
            }
            return;
        }
        
        //获取根目录下的所有文件
        File[] files = mRootDirectory.listFiles();
        if (files == null) {
            return;
        }
        for (File file : files) {
            try {
                long entrySize = file.length();
                CountingInputStream cis =
                        new CountingInputStream(
                                new BufferedInputStream(createInputStream(file)), entrySize);
                try {
                    CacheHeader entry = CacheHeader.readHeader(cis);
                    entry.size = entrySize;
                    putEntry(entry.key, entry);
                } finally {
                //流用完后要关闭
                    cis.close();
                }
            } catch (IOException e) {
            //发生异常，删除缓存文件
                file.delete();
            }
        }
    }

//使某个缓存项失效
@Override
public synchronized void invalidate(String key, boolean fullExpire) {
    Entry entry = get(key);
    //因为过期和刷新都是通过 ttl和softttl来判断的，所以可以通过设置属性来使缓存失效
    if (entry != null) {
        entry.softTtl = 0;
        if (fullExpire) {
            entry.ttl = 0;
        }
        //设置完属性后，将缓存放回缓存
        put(key, entry);
    }
}

@Override
public synchronized void put(String key, Entry entry) {
    if (mTotalSize + entry.data.length > mMaxCacheSizeInBytes
                && entry.data.length > mMaxCacheSizeInBytes * HYSTERESIS_FACTOR) {
            return;
        }
        File file = getFileForKey(key);
        try {
            BufferedOutputStream fos = new BufferedOutputStream(createOutputStream(file));
            CacheHeader e = new CacheHeader(key, entry);
            boolean success = e.writeHeader(fos);
            if (!success) {
                fos.close();
                VolleyLog.d("Failed to write header for %s", file.getAbsolutePath());
                throw new IOException();
            }
            fos.write(entry.data);
            fos.close();
            e.size = file.length();
            putEntry(key, e);
            pruneIfNeeded();
            return;
        } catch (IOException e) {
        }
        boolean deleted = file.delete();
        if (!deleted) {
            VolleyLog.d("Could not clean up file %s", file.getAbsolutePath());
        }
    }
    
 @Override
    public synchronized void remove(String key) {
        boolean deleted = getFileForKey(key).delete();
        removeEntry(key);
        if (!deleted) {
            VolleyLog.d(
                    "Could not delete cache entry for key=%s, filename=%s",
                    key, getFilenameForKey(key));
        }
    }
```

CacheHeader类是DiskBasedCache的一个静态内部类

```
//缓存头的大小
long size;

//标识缓存项的key
final String key;

//用于缓存一致性的ETag
final String etag;

//服务器日期
final long serverDate;

//最新修改的日期
final long lastModified;

//记录TTL
final long ttl;

//记录软TTL
final long softTtl;

//作用于缓存项的响应头，用list存储
final List<Header> allResponseHeaders;


private CacheHeader(
                String key,
                String etag,
                long serverDate,
                long lastModified,
                long ttl,
                long softTtl,
                List<Header> allResponseHeaders) {
            this.key = key;
            this.etag = "".equals(etag) ? null : etag;
            this.serverDate = serverDate;
            this.lastModified = lastModified;
            this.ttl = ttl;
            this.softTtl = softTtl;
            this.allResponseHeaders = allResponseHeaders;
        }
        
CacheHeader(String key, Entry entry) {
            this(
                    key,
                    entry.etag,
                    entry.serverDate,
                    entry.lastModified,
                    entry.ttl,
                    entry.softTtl,
                    getAllResponseHeaders(entry));
        }
        
 private static List<Header> getAllResponseHeaders(Entry entry) {
            //如果传进来的entry包含所有的响应头，则直接通过属性去使用
            if (entry.allResponseHeaders != null) {
                return entry.allResponseHeaders;
            }

            return HttpHeaderParser.toAllHeaderList(entry.responseHeaders);
        }

//从   CountingInputStream流中读取缓存头对象  
static CacheHeader readHeader(CountingInputStream is) throws IOException {
    int magic = readInt(is);
    if (magic != CACHE_MAGIC) {
        // don't bother deleting, it'll get pruned eventually
        throw new IOException();
    }
    String key = readString(is);
    String etag = readString(is);
    long serverDate = readLong(is);
    long lastModified = readLong(is);
    long ttl = readLong(is);
    long softTtl = readLong(is);
    List<Header> allResponseHeaders = readHeaderList(is);
    return new CacheHeader(
            key, etag, serverDate, lastModified, ttl, softTtl, allResponseHeaders);
}

//将数据转换成缓存项
Entry toCacheEntry(byte[] data) {
    Entry e = new Entry();
    e.data = data;
    e.etag = etag;
    e.serverDate = serverDate;
    e.lastModified = lastModified;
    e.ttl = ttl;
    e.softTtl = softTtl;
    e.responseHeaders = HttpHeaderParser.toHeaderMap(allResponseHeaders);
    e.allResponseHeaders = Collections.unmodifiableList(allResponseHeaders);
    return e;
}

//将缓存头的内容写入输出流
boolean writeHeader(OutputStream os) {
    try {
        writeInt(os, CACHE_MAGIC);
        writeString(os, key);
        writeString(os, etag == null ? "" : etag);
        writeLong(os, serverDate);
        writeLong(os, lastModified);
        writeLong(os, ttl);
        writeLong(os, softTtl);
        writeHeaderList(allResponseHeaders, os);
        os.flush();
        return true;
    } catch (IOException e) {
        VolleyLog.d("%s", e.toString());
        return false;
    }
}
```


CountingInputStream类是DiskBasedCache的一个静态内部类
> CountingInputStream 继承自FilterInputStream继承自 InputStream

```
//指定的长度
private final long length;

//记录已读的字节长度
private long bytesRead;

//传进来一个输入流，一个长度
CountingInputStream(InputStream in, long length) {
    super(in);
    this.length = length;
}

@Override
public int read() throws IOException {
    int result = super.read();
    if (result != -1) {
        bytesRead++;
    }
    return result;
}


@Override
public int read(byte[] buffer, int offset, int count) throws IOException {
    int result = super.read(buffer, offset, count);
    if (result != -1) {
    //result返回的是读取的字节长度
        bytesRead += result;
    }
    return result;
}

@VisibleForTesting
long bytesRead() {
    return bytesRead;
}

//字节空间留余
long bytesRemaining() {
    return length - bytesRead;
}
```







