**修改步骤：**

```
1.使用apktool反编译apk
一般命令： apktool d xx.apk 输出路径

2.修改apktool.yml中 sdkInfo：targetSdkVersion

3.回编译apk
一般命令： apktool b 输入文件 输出apk

4.签名apk
可使用360签名工具，比adb命令好用

5.查看apk的信息，如targetversion
方法一：使用jadx-gui，查看manifest中的targetversion
方法二：使用adb命令： aapt dump badging xx.apk
方法三：拖入android studio查看manifest
```
