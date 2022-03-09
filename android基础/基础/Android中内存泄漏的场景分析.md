产生原因：无法释放内存，最终导致内存溢出。

#### 静态变量
    
    如果静态变量持有某个Activity的context，而Activty销毁时无法被释放，最终会导致内存泄漏。（Application 的context不会存在这个问题）
    
#### 内部类
    
    非静态内部类持有外部类的引用。
    
    Handler的消息传递是串行的，当Activity被销毁回收时而Looper里面的消息还没处理完会导致内寸泄漏。因为ThreadLocal是静态的。
    
#### 资源对象未关闭
    
    Bitmap、Stream、Cursor、BroadcastReceiver等。
    解决办法：使用完后及时关闭。
    
    webview：让onDetachedFromWindow先走，在主动调用destroy()之前，把webview从它的parent上面移除掉。
    
#### 单例模式

    如果单例模式持有外部对象的引用，导致外部对象无法被释放，最终造成内存泄漏。