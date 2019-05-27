
RetryPolicy是一个接口类，提供了如下几个方法：

```
getCurrentTimeout()//获取当前超时时间

getCurrentRetryCount()//获取当前重试次数

retry(VolleyError error)//重试
```
