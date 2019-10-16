
##### RxJava的简单用法

```
//在gradle中添加依赖
implementation 'io.reactivex.rxjava2:rxjava:2.2.10'
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'


//实例化一个被观察者
Observable observable = Observable.create(new ObservableOnSubscribe() {
    @Override
    public void subscribe(ObservableEmitter emitter) throws Exception {
        emitter.onNext("msg1");
        emitter.onNext("msg2");
        emitter.onNext("msg3");
        emitter.onComplete();
    }
});

//实例化一个观察者
Observer<String> observer = new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        sendLog("onSubscribe");
    }
    @Override
    public void onNext(String s) {
        sendLog("onNext: " + s);
    }
    @Override
    public void onError(Throwable e) {
        sendLog("onError");
        e.printStackTrace();
    }
    @Override
    public void onComplete() {
        sendLog("onComplete");
    }
};

//绑定观察者与被观察者
observable.subscribe(observer);


//异步操作
observable.observeOn(AndroidSchedulers.mainThread())//
        .subscribeOn(Schedulers.io())
        .subscribe(observer);
```

