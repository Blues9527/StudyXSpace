### 主线程中的Looper.loop()一直无限循环为什么不会造成ANR？
    
> 首先，android是基于消息（事件）驱动的，在主线程中Looper会一直循环执行loop()方法从messageQueue里面取出消息交由handler去处理。如果主线程里面的事件没有及时得到处理（如looper被阻塞，或者需要处理的事件本身就是耗时操作），则会造成ANR。
    
> 另一种解读：真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长(dispatchMessage()中调用onCreate,onResume)，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。
    
> 主流的答案：
    
    1. Activity中输入事件超过5s没有响应
    2. BroadCastReceiver超过10s没有响应
    3. Service超过20s没有响应(没有启动)
    
### 如何避免ANR产生？
    
> 1.避免在主线程进行耗时操作（如网络请求，大量数据读写等），如需进行耗时操作可以开启一个子线程进行。
    
> 2.避免在broadcastReceiver中执行耗时操作（如网络请求，大量数据读写等）。