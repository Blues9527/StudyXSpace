
**RetryPolicy**是一个接口类，提供了如下几个方法：

```
getCurrentTimeout()//获取当前超时时间

getCurrentRetryCount()//获取当前重试次数

retry(VolleyError error)//重试
```

**DefaultRetryPolicy**类是RetryPolicy接口的一个实现类

构造方法
```
//无参构造，传入默认值initialTimeoutMs 为2.5s,maxNumRetries为1次，backoffMultiplier为 1
public DefaultRetryPolicy() {
        this(DEFAULT_TIMEOUT_MS, DEFAULT_MAX_RETRIES, DEFAULT_BACKOFF_MULT);
    }


@params initialTimeoutMs:策略初始化超时时间
@params maxNumRetries：最大重试次数
@params backoffMultiplier：超时时间的补偿乘积因子
public DefaultRetryPolicy(int initialTimeoutMs, int maxNumRetries, float backoffMultiplier) {
        mCurrentTimeoutMs = initialTimeoutMs;
        mMaxNumRetries = maxNumRetries;
        mBackoffMultiplier = backoffMultiplier;
    }
    
//获取当前超时时间
@Override
    public int getCurrentTimeout() {
        return mCurrentTimeoutMs;
    }
 
//获取当前重试次数  
@Override
    public int getCurrentRetryCount() {
        return mCurrentRetryCount;
    }
    
//获取补偿乘积因子
public float getBackoffMultiplier() {
        return mBackoffMultiplier;
    }
    
//RetryPolicy父类的一个实现方法
@Override
    public void retry(VolleyError error) throws VolleyError {
    //重试次数自增
        mCurrentRetryCount++;
        //当前超时时间（新） = 当前超时时间（旧）+当前超时时间（旧）*补偿乘积因子
        mCurrentTimeoutMs += (int) (mCurrentTimeoutMs * mBackoffMultiplier);
        //超过了最大重试次数则抛出异常
        if (!hasAttemptRemaining()) {
            throw error;
        }
    }

//如果当前重试的次数小于最大可重试次数，则返回true，反之亦然    
protected boolean hasAttemptRemaining() {
        return mCurrentRetryCount <= mMaxNumRetries;
    }
```

