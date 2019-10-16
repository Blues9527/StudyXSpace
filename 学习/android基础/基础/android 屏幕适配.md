1.屏幕分辨率适配
> 对每种屏幕分辨率都定义一套dimens.xml，可以使用screenmatch插件进行生成。
[ScreenMatch的安装与使用](https://blog.csdn.net/duanchuanzhi/article/details/81065011)

2.横竖屏适配
> 如果横竖屏内容变化过大建议横竖屏各用一套布局，在res下新建一个layout-land的文件夹，然后横竖屏命名一致系统就会识别了。

3.全面屏，刘海屏适配

[刘海屏适配](https://www.jianshu.com/p/561f7241153b/)
各大厂商都可以通过反射去获取是否有刘海屏
```
在application标签下添加以下属性
android:maxAspectRatio="2.4"
android:resizeableActivity="true"
android:supportsRtl="true"

添加以下元数据
        <!--全面屏-->
        <meta-data
            android:name="android.max_aspect"
            android:value="2.4" />
        <!--适配华为-->
        <meta-data
            android:name="android.notch_support"
            android:value="true" />
        <!--适配小米-->
        <meta-data
            android:name="notch.config"
            android:value="portrait|landscape" />
```
