
1.查看keystore签名内容命令：
> eg: keytool -list -v -keystore keystore文件路径

2.校验apk的签名版本（v1还是v2）：
> eg: apksigner verify -v apk路径

3.使用adb进行安装apk
> eg: adb install -r apk路径

4.查看包体信息，如targetversion，permission等
> eg: aapt dump badging apk路径

5.查看危险权限
> eg:adb shell pm list permissions -d -g