**1.1 操作系统相关**

1.什么是操作系统？

```
操作系统，也称OS（operating system），常见的操作系统有 windows、macOS和Linux等
```


2.什么是线程，什么是进程？

```
进程是操作系统资源分配的基本单位，线程是任务调度和执行的基本单位。

进程是线程的容器，每个进程至少有一条线程。
```

**1.2 JDK & JVM & JRE**

1.JDK & JVM & JRE分别是什么以及它们的区别

```
名词解释： JDK：Java Development Kit，即 Java开发包
           JVM：Java Visual Machine，即Java虚拟机
           JRE：Java Runtime Environment，即Java运行环境
           
    一般来说，JDK里面包涵Java源码及一些工具；
              JRE是保证Java代码能正常运行的基础；
              JVM是执行和编译Java代码的虚拟机。
```

2.解释一下为什么Java可以跨平台？

```
Java语言具有平台无关性主要是由于JVM会把对应的代码编译成对应的机器语言。
```


**1.3 面向过程 & 面向对象**

1.什么是面向过程 & 什么是面向对象，以及两者的区别？

```
转载： 原博文：https://blog.csdn.net/jerry11112/article/details/79027834

面向过程：
优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如单片机、嵌入式开发、 Linux/Unix等一般采用面向过程开发，性能是最重要的因素。 
缺点：没有面向对象易维护、易复用、易扩展

面向对象：
优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 更加灵活、更加易于维护 
缺点：性能比面向过程低
```


2.给我说说Java面向对象的特征以及讲讲你代码中凸显这些特征的经验。

```
Java面向对象的特性主要有：封装、继承和多态。
```


[3.什么是重载 & 什么是重写，以及两者的区别。](http://note.youdao.com/noteshare?id=92d67ea169c94dec1b3ebe17557173ec&sub=WEB3af821eaafb2b7fe97fb81ca8e0c1e76)

4.谈谈你对this和super的认识。

5.接口和抽象类的区别。

```
抽象类：使用abstract 关键字修饰，抽象类里面可以包含已实现的方法和未实现但是使用abstract关键字修饰的方法，子类继承抽象类的时候需要覆写父类所有抽象方法，若不覆写全部抽象方法则需要使用abstract声明为抽象。

接口：使用interface进行声明，接口里的方法不能进行实现，可以不用修饰符去修饰接口里的方法，默认为声明接口的修饰符。子类实现接口，需要重写接口里的所有方法，若不实现则编译时会报错。

抽象类和接口的区别：
    1.抽象类更适合作为基类存在，将所有共同特征抽象出来，让子类去继承，这样可以减少子类的代码。所以一般来讲，子类和抽象类具有某些共同的特征。
    2.接口则更像即插即用，子类可以与接口毫无共同特征。
    3.Java秉承单继承，多实现的原则。
```


6.静态属性和静态方法能被继承吗？静态方法又是否能被重写呢？

```
静态属性和方法无法被继承，也无法被重写
```


7.给我说说权限修饰符特性。

```
# private protected public default
private
被private修饰的方法只能在当前类使用，子承父类无法重写。

protected
被protected修饰的方法能在当前类，能在当前包下被调用，或跨包子承父类时被重写。

public
被public修饰的方法能跨包调用或者子承父类去被重写。 

default(没有修饰符)
没有修饰符修饰的方法，能在当前包下被调用，无法跨包子承父类时被重写。
```

8.给我谈谈Java中的内部类。

9.闭包和内部类的区别？

10.Java多态的实现机制是什么？

11.谈谈你对对象生命周期的认识？

12.static关键字的作用？

```
static是静态的意思，static可以作用于类、方法以及属性上。
作用于类时，只能在内部类声明时使用，声明的类为静态内部类，外部可以直接通过 外部类名.内部类名 去访问该类。
作用于方法时，声明出来的就是静态方法，外部可以直接通过 类名.方法名 去调用。
作用于属性的时候，表示是静态属性，外部可以直接通过 类名.属性名 去调用。
```

13.final关键字的作用。
```
final 关键字可作用于类、方法以及属性上。
当final作用于类的声明时，表示该类不能被继承。
当final作用于方法的时候，表示该方法不能被重写（无法被修改）。
当final作用于属性的时候，表示该属性无法被修改，即地址无法被改变。
```

**1.4 八大基本数据类型&引用类型**

1.说说Java中的8大基本类型 & 内存中占有的字节 & 什么是引用类型？

类型| 占有字节 | 多少位
---|---|---
char | 2| 16
byte | 1 | 8
short | 2 | 16
int |4| 32
long | 8 | 64
double | 8 | 64
float | 4 | 32
boolean |1/8 | 1


2.什么是拆箱 & 装箱，能给我举栗子吗？

原始类型 | 包装类型
---|---
char | Character
byte | Byte
short | Short
int | Integer
long | Long
double | Double
float | Float
boolean | Boolean

```
装箱是从原始类型向包装类型转换的过程；
拆箱是从包装类型向原始类型转换的过程。

这些过程都是JVM自动帮我们完成的，我们使用的时候可以直接使用如：

int i = Integer.parseInt(x);

```


**1.5 数组**

1.能说说多维数组在内存上是怎么存储的吗？

2.你对数组二次封装过吗？说说封装了什么

**1.6 Java异常**

1.说说Java异常体系主要用来干什么的 & 异常体系？

2.Error和Exception的区别？

3.说说运行时异常和非运行时异常的区别？

4.如何自定义一个异常？

5.throw和throws 的区别？

6.try{}catch{}finally{}可以没有finally吗？

7.finally语块有什么特点？

8.return在try{}catch{}finally{}中执行具有哪些规则？

9.给我例举至少5个常见的运行时异常。

**1.7 NIO/BIO/AIO**

1.NIO是什么 & BIO是什么 & AIO是什么 & 它们之间的区别？

2.IO按照方向和数据类型划分能划分为哪些数据流？

3.能给我说说NIO有什么特点？平常开发中使用过吗？

**1.8 集合(容器)**

1.说说Java中集合的框架？

2.Collection & Map区别

3.谈谈你常用的集合 & 它们底层的实现方式 & 优缺点 & 使用场景。

4.Map的遍历方式有哪些？

5.给我说说ArrayList的扩容机制.

6.什么是深拷贝 & 浅拷贝 & 如何深拷贝一个List集合.

7.Set是如何确保它的唯一性的。

8.你觉得HashMap的元素顺序和什么有关？

9.Java中HashMap如何解决哈希碰撞的？

10.ConcurrentHashMap如何实现并发访问的？

11.谈谈Java集合中那些线程安全的集合 & 实现原理。

12.说说有哪些集合能加入null,哪些不能加入null,为什么？

13.说说LinkedHashMap原理。

14.Collection 和 Collections的区别？

15.比较一下ArrayMap和HashMap。

16.说说HashMap的原理。

**1.9 线程**

1.什么是线程？能解决什么问题。

2.Java中创建线程的2种方式 & 区别？

```
注意：是创建线程而不是开启线程。

创建一个线程主要有两种方式：
    1.继承Thread类，重写run方法
    2.实现Runnable接口，重写run方法
    
还有一种比较少用，实现Callable和Future接口

```

3.给我说说线程的生命周期。

4.线程死锁的原因 & 举个栗子 & 如何避免死锁。

5.Synchronized放在静态方法和非静态方法上的锁对象分别是什么？

6.如何停止掉一个线程？

7.给我说说线程池的种类 & 特点 & 内部原理 & 平时当中使用案例。

8.给我谈谈你是如何保证线程数据安全问题的？

9.wait()和sleep()的区别？

10.什么是公平锁&非公平锁&区别？

11.给我讲讲线程间通信

12.volatile关键字是如何使用的？原理是什么

13.说说使用5个线程去计算一个数组之和的思路。

14.谈谈线程阻塞的原因有哪些？

15.谈谈你对notify的理解？

16.你觉得Lock和Synchronized的区别是什么？

17.谈谈你对ReentrantLock的认识。

[18.调用run()和start()的区别？](http://note.youdao.com/noteshare?id=161864297c118941142c42dbab078820&sub=WEB916c96a447c642d54afcb92102190ece)

```
run方法是Runnable接口里面的一个方法，实现了Runnable接口的类都需要实现该方法。run方法是线程里面的逻辑实现方法，主要用于处理业务逻辑。

start方法是Thread类的一个方法，Thread通过调用start方法去开进线程。
```


19.transient关键字的用法 & 作用 & 原理。

20.线程池的种类  & 工作原理 & ThreadPoolExecutor的工作策略有哪些？

21.ThreadLocal了解吗？说说原理。

22.权衡多线程的性能。

23.如何理解同步和异步，阻塞和非阻塞。

**1.10 泛型**

1.什么是泛型？能解决什么问题？

2.说说Java中泛型的工作机制？

3.在泛型种extends和super关键字的区别是什么？

```

```


**1.11 反射**

1.什么是反射？

2.如何获取一个类的成员变量 & 成员方法 & 注解信息 & ...。

```
属性
- getFields() //获取所有用public修饰的属性
- getDeclaredFields() //获取所有属性，包括public private protected default

方法
- getMethods() //获取所有用public修饰的方法，包括父类的方法
- getDeclaredMethods() //获取当前类的所有方法，包括public private protected

注解
- getAnnotations()//返回一个Annotation[]，如需获取具体的注解，需要做循环去拿到每一个annotation.annotationType()
```

3.通常在项目当中用到反射多吗？都是用来干嘛？
> 动态代理、DI（条目注入），butterknife、eventbus

**1.12 注解**

1.什么是注解 & 它和注释的区别？

```
注解（annotation），可用于类、方法以及属性。

标注属性：如butterknife
标注方法：如@override ，eventbus的@subscribe
标注类：如@controller
```


2.注解的工作机制是什么？

```
自定义注解的时候，可以指定注解生效的时机，如 SOURCE、CLASS和RUMTIME
```


**1.13 Socket编程**

1.什么是Socket编程？

2.什么是TCP,什么是UDP,二者之间区别如何？

```
TCP和UDP都是一种传输协议，位于OSI网络模型的传输层

TCP（Transmission Control Protocol），即传输控制协议，具有三次握手四次挥手的特性，是可靠的

UDP（User Datagram Protocol）,即用户数据报文协议，是不稳定，不可靠的，常用于直播行业
```


**1.14 设计模式**

1.说说设计模式的六大原则。

```
SOLID

单一职责原则（Single Responsibility Principle）

开关闭合原则（open closed principle）

里式替换原则（LSP liskov substitution principle）

迪米特原则（law of demeter LOD）

接口隔离原则（interface segregation principle）

依赖倒置原则（dependence inversion principle）
```

2.请讲讲你会使用的一些设计模式？

[3.请说说单例模式 & 你项目中常用的单例模式。](http://note.youdao.com/noteshare?id=62d7026aa87bae9f5b984c3084c90e87&sub=WEBbc28a46599f621e52c34dd58ac234933)

4.懒汉单例模式为什么要加volaitle？

5.能否给我说说Android中至少3个用到设计模式的栗子？

```
AlertDialog使用了 Builder模式
OkHttp 使用了Builder模式（OKHttpClient的构造函数）、代理模式、单例模式（SocketFactory）、工厂模式（SocketFactory）、简单工厂

BitmapFactory使用了工厂模式

```


**1.15 JVM相关**

1.什么是class文件？

2.Java代码执行流程？

3.Java内存结构 & 内存模型。

[4.GC回收机制。](http://note.youdao.com/noteshare?id=207909699a4e5738d958ec3836719f9d&sub=WEB67c3375591cea86f6740e3a5c072465b)

5.Java虚拟机是如何加载一个类的？

6.给我谈谈类加载器。

7.谈谈static编译运行时的流程，在虚拟机中如何保存的？

[8.说说Java种的4种引用以及用法？](http://note.youdao.com/noteshare?id=b48aaa5dbc51946b5f2f6fdd43d8a76b&sub=4694578443DA473CBD41115B3FD6FB37)

9.如何判断一个对象是死亡的？

10.代码中直接调用System.gc()会发生什么？

11.一个强引用直接被null赋值，那么这个对象会被立刻回收吗？

12.String a = "a"+"b"+"c";在内存中创建了几个对象？

13.谈谈你对字符集的理解。

14.常见的编码格式有哪些？

```
GBK，UTF-8
```

15.utf-8中的中文占几个字节？int型占几个字节？

16.谈谈你对逻辑地址和物理地址的理解？

17.你知道对象什么时候会回调finalize方法吗？

**1.16 其它Java部分有关面试题**

1.为什么局部内部类访问局部变量需要final?

2.String、StringBuffer、StringBuilder、CharSequence的区别。

```
String：适用于少量的字符串操作的情况

StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况

StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况

CharSequence是一个接口，String、StringBuilder和StringBuffer都实现了这个接口
```

3.equals和==的区别？

```

```


4.关于字符串的拼接你在项目中常常怎么操作的？为什么不能用“+”的方式进行拼接呢？

5.什么是Callback,讲讲你项目中使用的一些有关Callback的栗子。

6.retrun & break & continue 区别？

7.如何判断一个字符串是回文字符串？

8.final,finally,finalize的区别？

```
final是一个关键字，可作用于类、方法和属性，修饰后都不可被改变

finally，通常与try-catch使用，在finally代码块中进行资源释放和回收

finalize是object类的一个方法，执行finalize方法会在gc执行前调用该方法
```


9.什么是动态代理 & 什么是静态代理？

10.String为什么会加final？

```
String 是一个对象，一但生成则无法被改变。
```


11.OOM可以try{}catch{}吗？

12.给我谈谈正则表达式。

13.如何将String转成int?

14.谈谈你对String的理解。

15.你如何理解序列化？有哪些方式序列化？

16.谈谈你对依赖注入的理解。

17.给我谈谈你对分派的理解。