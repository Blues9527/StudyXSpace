
# private protected public default
- private
> 被private修饰的方法只能在当前类使用，子承父类无法重写。

- protected
> 被protected修饰的方法能在当前类，能在当前包下被调用，或跨包子承父类时被重写。

- public
> 被public修饰的方法能跨包调用或者子承父类去被重写。 

- default(没有修饰符)
> 没有修饰符修饰的方法，能在当前包下被调用，无法跨包子承父类时被重写。

# 反射
属性
- getFields() //获取所有用public修饰的属性
- getDeclaredFields() //获取所有属性，包括public private protected default

方法
- getMethods() //获取所有用public修饰的方法，包括父类的方法
- getDeclaredMethods() //获取当前类的所有方法，包括public private protected

# 事务

为了保证事务的正确执行，维护数据库的完整性，事务必须具有以下特性：
> 原子性(Atomicity)：事务的所有操作在数据库中要么都做，要么都不做。

> 一致性(Consistency)：事务的隔离执行(没有并发其他事务)保持数据库的一致性。

> 隔离性(Isolation)：一个事务内部操作和使用的数据对并发的其他事务是隔离的，并发事务之间互不影响。

> 持久性(Durability)：一个事务完成后，它对数据库的改变必须是永久性的，即使系统可能产生故障。

# 静态块、构造块、构造函数

> 静态代码块：最早执行，类被载入内存时执行，只执行一次。没有名字、参数和返回值，有关键字static。

> 构造代码块：执行时间比静态代码块晚，比构造函数早，和构造函数一样，只在对象初始化的时候运行。没有名字、参数和返回值。

> 构造函数：执行时间比构造代码块时间晚，也是在对象初始化的时候运行。没有返回值，构造函数名称和类名一致。

# for(;;)和while(true)区别
- 从寓意上来看，两种写法都是无限循环
- 从效率上看，while(true)每次循环要判断循环条件,for(;;)循环没有判断，理论上节省机器指令
 > 无论是for(;;)还是while(true),在Java中都是优化成goto没区别,结果来看,两种方法经过编译优化后,是一样的效果.

### String、StringBuilder&StringBuffer

String：适用于少量的字符串操作的情况

StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况

StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况