
# Socket

    socket实现了Closeable接口
    
    
### 构造函数


修饰符 | 参数列表| 异常
---|---|---
public | 空 | 无
public | Proxy proxy | 无
protected |SocketImpl impl | SocketException
public | String host, int port | 无
public | InetAddress address, int port | IOException
public | (String host, int port, InetAddress localAddr,int localPort| IOException
public | InetAddress address, int port, InetAddress localAddr, int localPort| IOException
public | String host, int port, boolean stream| IOException
public | InetAddress host, int port, boolean stream| IOException
private | InetAddress[] addresses, int port, SocketAddress localAddr,boolean stream | IOException

