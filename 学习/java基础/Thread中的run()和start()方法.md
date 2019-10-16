
# run()

> run()方法是Thread类实现Runnable接口时复写的方法。如果子类去继承Thread类，也必须重写run()方法，在run()方法里主要是编写业务逻辑。单纯地调用run()，也能实现业务逻辑，但是如果多个Thread都只是执行Thread.run()方法，执行结果是串行的，而不能实现真正意义上的多线程。

# start()
> start()方法是Thread类本身的一个方法，该方法有synchronized关键字修饰，意味着调用start()方法时，是能保证多线程安全的。在执行start()方法时，首先会判断当前线程的状态，如果是已经启动了的或者线程状态非0时，会抛出异常。然后会将该线程添加到ThreadGroup，最后会调用原生方法nativeCreate()[JDK 1.8]从而实现真正意义上的多线程并发。