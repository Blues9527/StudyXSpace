1.原因：

> 使用高版本的apktool进行反编译生成的yml文件里doNotCompress配置里存在某些特殊的文件导致无法进行压缩而反编译失败。（如：unity3d，lua等脚本文件）

2.如何解决

> 2.1 使用2.3.2及以下的apktool进行反编译，低版本如2.1.0或以下的apktool会出现反编译失败的情况。

> 2.2 在github上把对应版本的apktool源码拉取下来，然后修改brut.apktool -> apktool-lib ->src -> main ->java -> brut.android -> Androlib.java文件，然后找到 NO_COMPRESS_PATTERN 属性然后将无法压缩的文件格式添加进去。

```
//修改前
private final static Pattern NO_COMPRESS_PATTERN = Pattern.compile("\\.(" +
            "jpg|jpeg|png|gif|wav|mp2|mp3|ogg|aac|mpg|mpeg|mid|midi|smf|jet|rtttl|imy|xmf|mp4|" +
            "m4a|m4v|3gp|3gpp|3g2|3gpp2|amr|awb|wma|wmv|webm|mkv)$");
//修改后
private final static Pattern NO_COMPRESS_PATTERN = Pattern.compile("\\.(" +
            "jpg|jpeg|png|gif|wav|mp2|mp3|ogg|aac|mpg|mpeg|mid|midi|smf|jet|rtttl|imy|xmf|mp4|" +
            "m4a|m4v|3gp|3gpp|3g2|3gpp2|amr|awb|wma|wmv|webm|mkv|unity3d|lua)$");
```
