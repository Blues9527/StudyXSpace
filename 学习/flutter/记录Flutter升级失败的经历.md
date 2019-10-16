1.升级起因：
> flutter老提示我 flutter upgrade ，忍不住手贱去升级了。
    
升级尝试&问题解决：
> 1.我看到flutter是有git管理的，直接git pull，但是升级无效

> 2.使用cmd命令行 先flutter doctor，然后flutter upgrade，但是升级后再flutter doctor就有错误提示了。

> 3.此前错误实践，只能把之前的flutter删掉然后重新配置环境了，然后首先在github上拉，拉下来master然后切换分支，但是发现里面竟然没有flutter-sdk和dart-sdk，踩坑了。

> 4.然后只能去官网下载了，然后终于以为没问题了，然而，打开旧项目，代码飘红，提示的是无法识别源码路径（我下载后 文件解压出来的有两层文件夹，我保留了，然后修改了一下环境变量的路径），但是一运行项目就报错了，看了提示还是旧的packages提示。

```
提示如下: 

running xxx : 当前配置flutter/bin路径

previous xxx： 之前配置的flutter/bin路径

```


> 5.修改.packages的路径，然后首次运行还是报错但是项目能正常运行起来。然后终于ok了。