因为kotlin没有static这一关键字，所以不能像Java那样声明为静态，但是kotlin中有 companion关键字，companion翻译过来即为半生，通常配合 object使用，即 companion object{} 即为伴生对象

```
ex1：
class Singleton private constructor() {

    companion object {
        val getInstance = SingletonHolder.holder
    }

    private object SingletonHolder {
        val holder = Singleton()
    }
}

用法：
Singleton.getInstance
```


```
ex2：
class SingletonForKotlin private constructor() {

    companion object {
        val instance: SingletonForKotlin? = SingletonForKotlin()
    }

}

用法：
SingletonForKotlin.instance
```


```
ex3：类似Java的 DCL双判空
class SingletonForJava private constructor() {
    companion object {

        @Volatile
        private var INSTANCE: SingletonForJava? = null

        val instance: SingletonForJava
            get() {
                if (INSTANCE == null) {
                    synchronized(SingletonForJava::class.java) {
                        if (INSTANCE == null) {
                            INSTANCE = SingletonForJava()
                        }
                    }
                }
                return INSTANCE!!
            }
    }
}

用法：
SingletonForJava.instance
```


