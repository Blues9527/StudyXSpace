# Apk的优化
- 使用svg替代多分辨率套图

> 导入方式 new -> Vetor Asset

> vertorDrawables.useSupportLibaray=true;

- 使用tint着色器代替多颜色套图

> tintMode  

- String.xml资源配置(android-support-v7包会生成很多种语言)

> 配置resConfigs "zh"

- so库配置，减少无关库。

> 配置 ndk{ abiFilters 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'x86' }//设置支持的SO库架构,一般手机都是这三个

- 移除无用resource资源 

> 配置shrinkResources true

- 源代码混淆

> 配置 minifyEnabled true

> 配置proguardFile 文件，默认是 proguardFile getDefaultProguardFile()

> 原理是：之前的方法名或者类名都是按照功能或者需求去命名的，混淆后会被替换成简单的字母abc之类的，减少了代码体积。

- 资源压缩

- 使用webp替代png/jpg

> 如项目之前有png或者jpg格式图片，右击该图片点击 convert to WebP... 会生成一张webp文件图片

- 资源混淆/压缩对齐(apk)

> 配置 zipAlignEnabled true

### 参考文章
[Apk不得不看的瘦身大全](https://juejin.im/post/5da1486951882505202587b8)