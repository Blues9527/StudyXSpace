1.popupwindow显示不全
> 设置popupWindow.setClippingEnabled(false);

2.手机横屏时，软键盘铺满屏幕遮挡住EditText
> EditText.setImeOptions(EditorInfo.IME_FLAG_NO_EXTRACT_UI);

> android:imeOptions="flagNoExtractUi"//设置了好像没效果，有待商榷