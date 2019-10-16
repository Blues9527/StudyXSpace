[参考@zhwadezh](https://blog.csdn.net/zhwadezh/article/details/79310119)

> Binder通信采用C/S架构

> Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次

> Binder机制从协议本身就支持对通信双方做身份校检，因而大大提升了安全性

### Binder通信的四个角色

Client进程：使用服务的进程。

Server进程：提供服务的进程。

ServiceManager进程：ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。

Binder驱动：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

### Binder运行机制

注册服务(addService)：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。

获取服务(getService)：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。

使用服务：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。
