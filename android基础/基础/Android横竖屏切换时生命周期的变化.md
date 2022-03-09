
### 正常启动时的生命周期

    onCreate()、onStart()和onResume()

1.竖屏切换至横屏/横屏切换至竖屏
    
    onPause()->onStop()->onSaveInstanceState()->onDestroy()->onCreate()->onStart()->onRestoreInstanceState()->onResume()

2.如果在manifest中配置了configChanges="orientation|keyboardHidden|screenSize"
则只会调用
    
    onConfigurationChanged()
    
3.如果关闭了屏幕/置于后台/跳转到另一个activity时,则会调用
    
    onPause()、onStop()和onSaveInstanceState(),
亮屏时会调用/回到前台时/由另一个activity返回时
    
    onRestart()、 onStart()和onResume()
    
4.如果直接清空后台，则会调用
    
    onPause()、onStop()、onSaveInstanceState()和onDestroy()方法
    