
```
首先看一下TextField的入口函数,相当之多的属性，不过都是可选属性
 const TextField({
    Key key,
    this.controller,//控制器，通过控制器可以获取textfield的内容
    this.focusNode,//焦点
    this.decoration = const InputDecoration(),//装饰器
    TextInputType keyboardType,//键盘输入类型，如数字，文本等
    this.textInputAction,//更改手机软键盘的显示内容
    this.textCapitalization = TextCapitalization.none,//大写相关，如首字母大写，全部大写
    this.style,
    this.strutStyle,
    this.textAlign = TextAlign.start,//文本对齐方式，默认从左到右
    this.textAlignVertical,//文本垂直对齐
    this.textDirection,
    this.readOnly = false,//是否只读
    this.showCursor,//是否显示光标
    this.autofocus = false,//自动获取焦点
    this.obscureText = false,//模糊（文本）提示
    this.autocorrect = true,//自动更正
    this.maxLines = 1,//最小行数默认是1
    this.minLines,//最小行数
    this.expands = false,//是否可以扩展
    this.maxLength,//最大长度
    this.maxLengthEnforced = true,
    this.onChanged,//textfield内容改变监听
    this.onEditingComplete,//textfield编辑完成监听
    this.onSubmitted,//提交监听
    this.inputFormatters,
    this.enabled,//是否可用
    this.cursorWidth = 2.0,//光标宽度
    this.cursorRadius,//光标弧度
    this.cursorColor,//光标颜色
    this.keyboardAppearance,
    this.scrollPadding = const EdgeInsets.all(20.0),
    this.dragStartBehavior = DragStartBehavior.start,
    this.enableInteractiveSelection,
    this.onTap,//点击事件
    this.buildCounter,//数字计数器
    this.scrollController,//滚动控制器
    this.scrollPhysics,//物理滚动
  })
```


```

```

