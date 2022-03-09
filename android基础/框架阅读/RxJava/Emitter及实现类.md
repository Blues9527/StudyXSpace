Emitter<T>是一个泛型接口

```
onNext能发送多个
onError与onComplete存在互斥关系，如同时发送只有一个能生效

//发送正常值
 void onNext(@NonNull T value);
 
 //发送异常标识
 void onError(@NonNull Throwable error);
 
 //发送完成标识
 void onComplete();
```

ObservableEmitter<T>是Emitter<T> 的子类
```
public interface ObservableEmitter<T> extends Emitter<T> {

    void setDisposable(@Nullable Disposable d);

    void setCancellable(@Nullable Cancellable c);

    boolean isDisposed();

    @NonNull
    ObservableEmitter<T> serialize();
    
    
    boolean tryOnError(@NonNull Throwable t);
}
```
