### 1.混淆(proguard)

```
混淆是在打包的时候，通过替换类名和方法名去防止随意地去查阅源码甚至是通过反编译去修改源码。

混淆的好处：
1.提高他人阅读源码的难度
2.具有一定压缩包体体积的效果
```


### 2.混淆配置

```
build.gradle中
在buildTypes{
    minifyEnabled true //配置为true则执行proguard混淆
    proguardFiles getDefaultProguardFile('proguard-android.txt')//默认配置混淆规则
    proguardFile 'xxx.pro'//额外配置混淆文件，可以配置多个
}

```
**xx.pro文件中的一些常用配置**

**Input/Output options(输入输出选项)**
- @filename //是 -include filename 的简写
- -include filename
- -basedirectory directoryname
- -injars class_path
- -outjars class_path
- -libraryjars class_path
- -skipnonpubliclibraryclasses
- -dontskipnonpubliclibraryclasses
- -dontskipnonpubliclibraryclassmembers
- -keepdirectories [directory_filter]
- -target version
- -forceprocessing

**Keep options(保留选项)**
- -keep [,modifier,...] class_specification

> 指定要保存为代码入口点的类和类成员(属性和方法)。例如，为了保存应用程序，可以指定main类及其main方法。为了处理库，您应该指定所有可公开访问的元素。

- -keepclassmembers [,modifier,...] class_specification

> 指定要保存的类成员(如果它们的类也被保存)。如保留实现可序列化接口的类的所有序列化字段和方法。

- -keepclasseswithmembers [,modifier,...] class_specification

> 指定要保存的类和类成员，条件是所有指定的类成员都存在。例如，您可能希望保留所有具有主方法的应用程序，而不必显式地列出它们。

- -keepnames class_specification

> 是 -keep allowshrinkingclass_specification的简写，指定要保留其名称的类和类成员(如果在压缩阶段中没有删除它们)。例如，您可能希望保留实现可序列化接口的类的所有类名，以便处理后的代码与任何初始序列化的类保持兼容。完全不使用的类仍然可以删除。只适用于混淆。

- -keepclassmembernames class_specification

> 是 -keepclassmembers allowshrinkingclass_specification的简写，指定要保留其名称的类成员(如果在压缩阶段中没有删除它们)。例如，在处理JDK 1.2或更高版本编译的库时，您可能希望保留合成类$methods的名称，这样当处理使用已处理库的应用程序时，混淆器可以再次检测到它(尽管ProGuard本身不需要这样做)。只适用于混淆。

- -keepclasseswithmembernames class_specification

> 是-keepclasseswithmembers allowshrinkingclass_specification的简写，指定要保留其名称的类和类成员，条件是所有指定的类成员都在压缩阶段之后出现。例如，要保留所有native方法名称及其类的名称，以便处理后的代码仍然可以链接到native库。根本不使用的native方法仍然可以删除。如果使用了一个类文件，但是没有使用它的native方法，那么它的名称仍然会被混淆。只适用于混淆。

- -if class_specification

> 指定必须存在的类和类成员，以激活后续的keep选项(-keep，-keepclassmembers，…)。该条件和后续的keep选项可以共享通配符和对通配符的引用。例如，您可以使用Dagger和Butterknife之类的框架，在项目中存在具有相关名称的类的情况下保留类。

- -printseeds [filename]

> 指定详细列出由各种-keep选项匹配的类和类成员。列表被打印到标准输出或给定文件中。该列表对于验证是否找到了预期的类成员非常有用，特别是在使用通配符时。例如，您可能想列出所有的应用程序或您正在保存的所有applet。

**Shrinking options(压缩选项)**
- -dontshrink
> 指定不压缩代码。默认情况下，ProGuard会进行压缩代码操作，即删除所有未使用的类和类成员，只保留各种使用-keep列出的选项，以及直接或间接依赖的选项。它还在每个优化步骤之后应用一个压缩步骤，因为一些优化可能会打开删除更多类和类成员的可能性。

- -printusage [filename]

> 指定列出输入类文件的死代码。列表被打印到标准输出或给定文件中。例如，可以列出应用程序中未使用的代码。仅适用于收缩。
 
- -whyareyoukeeping class_specification

> 指定打印有关为什么在收缩步骤中保留给定类和类成员的详细信息。如果想知道为什么某个给定的元素会出现在输出中，那么这将非常有用。总的来说，有很多不同的原因。此选项为每个指定的类和类成员将最短的方法链打印到指定的种子或入口点。在当前实现中，打印出来的最短链有时可能包含循环扣除，但这些扣除并不反映实际的缩减过程。如果指定-verbose选项，则跟踪包含完整字段和方法签名。仅适用于收缩。

**Optimization options(优化选项)**
- -dontoptimize
> 指定不优化输入类文件。默认情况下，ProGuard优化所有代码。它内联并合并类和类成员，并在字节码级别优化所有方法。

- -optimizations [optimization_filter]
> 指定要在更细粒度级别上启用和禁用的优化。只适用于优化。这是一个专家的选择。

- -optimizationpasses n
> 指定要执行的优化传递数。默认情况下，只执行一次传递。多次通过可能会导致进一步的改进。如果优化通过后没有发现任何改进，则结束优化。只适用于优化。

- -assumenosideeffects class_specification
- -assumenoexternalsideeffects class_specification
- -assumenoescapingparameters class_specification
- -assumenoexternalreturnvalues class_specification
- -assumevalues class_specification
- -allowaccessmodification
- -mergeinterfacesaggressively

**Obfuscation options(混淆选项)**
- -dontobfuscate
> 指定不混淆输入类文件。默认情况下，ProGuard混淆代码:它为类和类成员分配新的短随机名称。它删除了仅对调试有用的内部属性，如源文件名、变量名和行号。

- -printmapping [filename]
- -applymapping filename
- -obfuscationdictionary filename
- -classobfuscationdictionary filename
- -packageobfuscationdictionary filename
- -overloadaggressively
- -useuniqueclassmembernames
- -dontusemixedcaseclassnames
- -keeppackagenames [package_filter]
- -flattenpackagehierarchy [package_name]
- -repackageclasses [package_name]
- -keepattributes [attribute_filter]
- -keepparameternames
- -renamesourcefileattribute [string]
- -adaptclassstrings [class_filter]
- -adaptresourcefilenames [file_filter]
- -adaptresourcefilecontents [file_filter]

**Preverification options(预校验选项)**
- -dontpreverify
- -microedition
- -android

 **General Options(通用选项)**
- -verbose
> 指定在处理期间写出更多信息。如果程序因异常而终止，此选项将打印出整个堆栈跟踪，而不只是异常消息。

- -dontnote [class_filter]
> 指定不打印有关配置中潜在错误或遗漏的注释，例如类名中的拼写错误或缺少可能有用的选项。可选的过滤器是一个正则表达式;ProGuard不打印具有匹配名称的类的注释。

- -dontwarn [class_filter]
> 指定根本不警告未解决的引用和其他重要问题。可选的过滤器是一个正则表达式;ProGuard不会打印关于具有匹配名称的类的警告。忽视警告可能是危险的。例如，如果处理确实需要未解析的类或类成员，则处理后的代码将不能正常工作。只有当你知道你在做什么的时候才使用这个选项!

- -ignorewarnings
>指定打印关于未解决引用和其他重要问题的任何警告，但在任何情况下都要继续处理。忽视警告可能是危险的。例如，如果处理确实需要未解析的类或类成员，则处理后的代码将不能正常工作。只有当你知道你在做什么的时候才使用这个选项!

- -printconfiguration [filename]
> 指定写出已解析的整个配置，包括文件和替换的变量。结构被打印到标准输出或给定文件中。有时，这对于调试配置或将XML配置转换为更可读的格式非常有用。

- -dump [filename]
> 指定在任何处理之后写出类文件的内部结构。结构被打印到标准输出或给定文件中。例如，您可能想写出给定jar文件的内容，而不需要处理它。

- -addconfigurationdebugging



### 3.[混淆规则](https://www.guardsquare.com/en/products/proguard/manual/usage#keepoverview)
可参考官方规则

```
关键字
1.class 包含类和接口
2.enum 枚举
3.$ 使用与内部类 如 :-keep public class com.blues.a$b
```

```
[@annotationtype] [[!]public|final|abstract|@ ...] [!]interface|class|enum classname
    [extends|implements [@annotationtype] classname]
[{
    [@annotationtype]
    [[!]public|private|protected|static|volatile|transient ...]
    <fields> | (fieldtype fieldname [= values]);

    [@annotationtype]
    [[!]public|private|protected|static|synchronized|native|abstract|strictfp ...]
    <methods> | <init>(argumenttype,...) | classname(argumenttype,...) | (returntype methodname(argumenttype,...) [return values]);
}]
```

**包名的匹配规则及通配符**

通配符| 描述
---|---
? | 匹配任意单个字符，不包括包分隔符(.);如：com.example.a?可匹配 com.example.a1,com.example.a2,但不能匹配 com.example.a12
* |匹配任意部分类名，但是不包括包分隔符(.);如，com.example.\*Test\*可匹配com.example.MyTestClass 但是不能匹配单边的如com.example.MyTest
** | 匹配任意部分类名，可以包含包分隔符;如**.Test可以匹配所有非root package下的Test类，或者是com.example.**可以匹配所有com.example下的类和com.example子包下的所有类
<n> | 匹配同一选项中第n个匹配的通配符，如 com.example.*Foo<1> 可以匹配 com.example.BarFooBar


**属性和方法能通过特殊的通配符去匹配**
配置 | 描述
---|---
<init> | 匹配任意构造函数
<fields>| 匹配任意属性
<methods>| 匹配任意方法
*| 匹配任意方法或属性


**属性和方法也能通过特定的规则表达式去匹配，如：**
配置 | 描述
---|---
? | 匹配方法名称中的任何单个字符
* | 匹配方法名称的任何部分
<n> | 匹配同一选项中第n个匹配的通配符

**描述符中还可以包含如下通配符**

header 1 | header 2
---|---
% | 匹配任意初始类型，如boolean，int而不是void
? | 匹配类名中的任意单个字符
* | 匹配类名中不包含包分隔符的任何部分
** | 匹配类名的任意部分，可能包含任意数量的包分隔符。
*** | 匹配任意类型（包括原始的或非原始的，数组或者发给数组）
... | 匹配任何类型的任意数量的参数
<n> | 匹配同一选项中第n个匹配的通配符



---


### 4.混淆中常用的配置
- 自定义view控件类不能被混淆
- 实现了Serializable 和 Parcelable的类（即实现了序列化的类）不能被混淆
```
-keep class * implements android.os.Parcelable{
    public static final android.os.Parcelable$Creator *;
}

-keepclassmembers class * implements java.io.Serializable {
   static final long serialVersionUID;
   private static final java.io.ObjectStreamField[] serialPersistentFields;
   !static !transient <fields>;
   private void writeObject(java.io.ObjectOutputStream);
   private void readObject(java.io.ObjectInputStream);
   java.lang.Object writeReplace();
   java.lang.Object readResolve();
}
```
- 如果有外放接口，也不能被混淆
- 枚举不能被序列化
```
-keepclassmembers enum  * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```
- 被@JavascriptInterface注解的方法不能被混淆//主要是与js交互时是通过调用方法名去实现交互的
```
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}
```
- R文件及R文件下的所有类和方法都不能被混淆

```
-keep class **.R$* {*;}
-keepclassmembers class **.R$* {*;}
```
- BuildConfig文件不能被混淆

```
-keep class **.BuildConfig {*;}
```

-jni/native不混淆

```
-keepclasseswithmembernames class * {
    native <methods>;
}
```
- 泛型不混淆

```
-keepattributes Signature
```


