1.popupwindow显示不全
> 设置popupWindow.setClippingEnabled(false);

2.手机横屏时，软键盘铺满屏幕遮挡住EditText
> EditText.setImeOptions(EditorInfo.IME_FLAG_NO_EXTRACT_UI);

> android:imeOptions="flagNoExtractUi"//设置了好像没效果，有待商榷

3.intent携带跳转的最大数据是多大？
> BINDER_VM_SIZE=（1*1024*1024）-（4096*2）也就是小于1M

4.as通过run会生成一个debug.apk，直接给用户安装时无法安装的。

```
原因：因为直接run会在manifest文件中添加一个属性 android:testOnly=”true”，使得apk无法被安装。

强制安装：使用adb install -t + x.apk可以强制安装

解决办法：在gradle.properties中设置 android.injected.testOnly=false
```
