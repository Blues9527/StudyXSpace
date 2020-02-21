[toc]

### 1.Java中的class loader

#### 1.1 class loader 类型
> Java中包含4中类型的class loader，分别是 Boostrap ClassLoader / Extensions ClassLoader / Application ClassLoader /自定以ClassLoader

#### 1.2 java程序启动时，都会用到哪些class loader呢？
> Boostrap ClassLoader / Extensions ClassLoader / Application ClassLoader

#### 1.3双亲委托机制
> 双亲委托机制的原理是：当要加载某个类的时候，先查找类是否已经加载过，如果没有加载过则去委托父类的类加载器，然后逐层委托直到顶层boostrap class loader，如果顶层类加载器没有查找出对应的类，则逆向往回逐层的类加载器进行查找，如果都没有查找到，则抛出异常，找到则直接返回。

#### 1.4双亲委托机制的优点是什么？
> 避免自定义与系统相同的类加载的问题。双亲委托机制可以保障优先加载系统的类而不是自定义的类，则避免直接修改系统类的问题。

#### 1.5 双亲委托机制核心代码

```
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
{
        //首先会查找是否已加载过该类
        Class<?> c = findLoadedClass(name);
        //为空，没有加载过
        if (c == null) {
            try {
            //判断父类加载器是否为空
                if (parent != null) {
                //不为空调用父类加载器loadclass方法
                    c = parent.loadClass(name, false);
                } else {
                //父类加载器为空，查找BootStrapClassLoader是否有加载过
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            //如果还是没找到，跑出异常
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                c = findClass(name);
            }
        }
        //如果已经加载过了直接返回该类
        return c;
}

protected final Class<?> findLoadedClass(String name) {
    ClassLoader loader;
    if (this == BootClassLoader.getInstance())
        loader = null;
    else
        loader = this;
        //虚拟机类加载器查找
    return VMClassLoader.findLoadedClass(loader, name);
}
```


### 2.Android中的class loader