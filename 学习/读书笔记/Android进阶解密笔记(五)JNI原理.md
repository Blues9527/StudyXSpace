1.JNI是什么？为什么需要JNI？
> JNI的全称是 java native interface，是Java层与native 层交互的桥梁。通过JNI去调用native方法，而native方法很多都是c/c++编写的，在效率方面会更高一点，所以在很多场景下都会用到JNI，比如说游戏开发、音视频解码编码，热修复插件化等。

2.native方法注册
> native方法注册可以分为静态注册和动态注册两种方法。

> 静态注册：根据方法名将Java方法和JNI函数建立关联。

> 动态注册：通过JNIEnv的RegisterNatives完成动态注册。


```
//JNINativeMethod
typedef struct{
    const char* name;//Java方法名字
    const char* signature;//Java方法的签名信息
    void* fnPtr;//JNI中对应的方法指针
} JNINativeMethod;
```


3.基本数据类型的转换

Java | Native | Signature
---|---|---
byte | jbyte | B
char | jchar | C
double | jdouble | D
float | jfloat | F
int | jint | I
short | jshort | S
long | jlong | J
boolean | jboolean | Z
void | void | V

4.引用数据类型转换

Java | Native | Signature
---|---|---
对象 | jobject | L+classname+;
Class | jclass | Ljava/lang/Class;
String | jstring | Ljava/lang/String;
Throwable | jthrowable | Ljava/lang/Throwable;
Object[] | jobjectArray | [L+classname+;
byte[] | jbyteArray | [B
char[] | jcharArray | [C
double[] | jdoubleArray | [D
float[] | jfloatArray | [F
int[] | jintArray | [I
short[] | jshortArray | [S
long[] | jlongArray | [J
boolean[] | jbooleanArray | [Z

5.JNI方法签名的格式

```
//通过;去划分参数格式签名，最后也以;结束
(参数签名格式;参数签名格式;...;)返回值签名格式

举个栗子：
private native final void native_setup(Object mediarecorder_this,String clientName,String opPackageName) throws IllegalStateException;

对应的方法签名是:
(Ljava/lang/Obkect;Ljava/lang/String;Ljava/lang/String;)V
```

6.通过命令生成方法签名

```
//将.java文件编译成 .class文件
javac 目标文件.java

//生成方法签名
//s表示输出内部类型签名，p表示打印出所有的方法和成员(默认打印public成员)
javap -s -p 目标文件.class
```

![image](https://github.com/Blues9527/BluesStudying/blob/master/image/%E7%94%9F%E6%88%90%E6%96%B9%E6%B3%95%E7%AD%BE%E5%90%8D.png?raw=true)

7.JNIEnv

```C++
struct _JNIEnv{
    const JNINativeInterface* functions;
#if defind(_cplusplus)
    ...
    //Findclass方法去找到java层对应的类
    jclass FindClass(const char* name)
    {return functions->FindClass(this,name);}
    
    ...
    //GetMethodID方法获取java层的方法
    jmethodID GetMethodID(jclass clazz,const char* name,const char* sig)
    {return functions->GetMethodID(this,clazz,name,sig);}
    
    ...
    //GetFieldID方法获取java层的属性
    jfieldID GetFieldID(jclass clazz,const char* name,const char* sig)
    {return functions->GetFieldID(this,clazz,name,sig);}
    
    ...
}
```

8.JNI的引用类型
- 本地引用(LocalReferences是JNI中最常见的引用类型)
- 全局引用(GlobalReferences)
- 弱全局引用(WeakGlobalReferences)

8.1 本地引用
特点
- 当native函数返回时，这个本地引用就会自动被释放
- 只在创建它的线程中有效，不能够跨线程使用
- 局部引用是JVM负责的引用类型，受JVM管理

```
可以使用DeleteLocalRef来手动删除本地引用
```

8.2 全局引用
特点
- 在native函数返回时不会自动释放，因此全局引用需要手动来进行释放，并不会被GC回收
- 全局引用是可以跨线程使用的
- 全局引用不受到JVM管理

```
使用NewGlobalRef创建，使用DeleteGlobalRef释放
```

8.3 弱全局引用
特点
> 特点与全局引用相似，不同的是可以被GC回收

```
使用NewWeakGlobalRef创建，使用DeleteWeakGlobalRef释放
```



