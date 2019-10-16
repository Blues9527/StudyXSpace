**1.var 与 val**
var： var是一个可变变量，这是一个可以通过重新分配来更改为另一个值的变量。这种声明变量的方式和Java中声明变量的方式一样。

val: val是一个只读变量，这种声明变量的方式相当于java中的final变量。一个val创建的时候必须初始化，因为以后不能被改变。


```
val  创建全局变量时，必须需要初始化，当生命局部变量时，可以不赋值，但是需要声明初始类型
```


```
var nameVar : String = "Blues"
val nameVal : String = "Lam"

fun testPrint(){
    println(nameVar)
    println(nameVal)
    
    nameVar = "Blues1"
    printl(nameVar)
    
    nameVal= "Lam1" (AS报错:val can not be reassigned，即不能被改变)
}
```

**2.拓展 const val 与 val**
- const val只能在 top-level、object和companion object的成员中声明
- 只允许String或者原始属性初始化
- 不能自定义get()


```
同样是声明字符串，const val是直接赋值的，而 val是是通过get方法去获取值的，所以const val效率更好。

const val 和 val 都是编译成 public static final
```


