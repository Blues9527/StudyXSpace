##### 1.一个apk包体里包含哪些东西？

一般apk里面包含以下内容：
1. lib（存放so文件的）
2. res（开发时的资源）
3. META-INF（证书、签名之类的，以及很多.version文件）
4. resources.arsc（打包后的资源的文件）
5. assets文件
6. AndroidManifest.xml文件（存放权限，组件，元数据等）
7. dex文件（java文件经过编译打包后生成适用于davlik虚拟机读取的文件）





[APK打包流程](https://blog.csdn.net/loongago/article/details/89646920) 
这篇文章讲的十分详细