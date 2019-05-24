#### 1.什么是JavaBean？
> JavaBean是一个特殊的类,用Java语言书写,并遵循JavaBean API规范.

#### 2.如何构造一JavaBean对象？
一般有如下几种方法：
> 1）通过构造器（构造函数）去实例化一个对象。即在构造函数中传入参数，最简单的一种。

> 2.1）通过JavaBean 空构造实例化对象后，通过setter方法去设置参数。

> 2.2) 同JavaBean上述方法的优化版，通过链式结构去设置参数。即setter方法中返回实力类对象本身，一般是return this；

> 3) 可通过Builder的方式去构造一个实体类。即在实体类内部创建一个静态内部Builder类，在外面实力类对象的构造函数设为私有的并传进去一个Builder对象，然后将实力类的属性和builder类的属性保持一致，这样就可以通过构造Builder去构造一个实体类对象，使用的时候是实例化一个builder对象，因为Builder类里面有个build方法，最后通过build方法去返回实体类对象，并传builder对象进去。

###### 通过builder方式的demo

```
public class Test {

    private Test test;
    private String arg1, arg2;

    private Test(Builder builder) {
        this.arg1 = builder.arg1;
        this.arg2 = builder.arg2;
    }

    public String arg1() {
        return arg1;
    }

    public String arg2() {
        return arg2;
    }

    public static class Builder {
        String arg1, arg2;

        public Builder() {

        }

        Builder(Test test) {
            this.arg1 = test.arg1;
            this.arg2 = test.arg2;
        }

        public Builder params1(String arg1) {
            this.arg1 = arg1;
            return this;
        }

        public Builder params2(String arg2) {
            this.arg2 = arg2;
            return this;
        }

        public Test build() {
            return new Test(this);
        }

    }

    @Override
    public String toString() {
        return "args1: " + this.arg1 + " arg2: " + this.arg2;
    }
}
```

#### 各种方式的对比
1.采用构造器方式：不够灵活，但是最简单的。

2.采用JavaBean方式：无法保证对象的状态一致性，多线程下需要确保线程安全。

3.采用Builder方式：代码可读性提高，创建后对象无法被改变，但是写起来比较复杂。

