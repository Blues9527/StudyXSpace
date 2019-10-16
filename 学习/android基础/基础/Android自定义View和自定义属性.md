### Android自定义view

- View的绘制流程
> 测量(performMeasure) -> 布局(performLayout) -> 绘制(performDraw) , 对应view或viewgroup里面的就是 onMeasure,onLayout,onDraw

- 自定义view，需要继承自view，viewgroup或一些特定的view。
> 自定义view主要覆写3个方法 onMeasure/onLayout/onDraw
> 主要涉及到的类有Paint(画笔) Canvas(画布) Rect(矩形 int) RectF(矩形 float)

- View中的 MeasureSpec

```
MeasureSpec中有三种测量模式: MeasureSpec.EXACTLY | MeasureSpec.UNSPECIFIED | MeasureSpec.AT_MOST

UNSPECIFIED：不指定测量模式

EXACTLY：精确测量模式

AT_MOST：最大值模式
```

 



### 自定义属性 

> 需要创建一个attrs.xml文件，里面存放自定义属性。最外层用<declare-styleable name="">声明，用<attr name="" format/>声明属性。然后在自定义view类中的构造方法用context.obtainStyledAttributes找到置顶的styleable文件，并用TypedArray进行存储。属性通过 TypedArray中的各种get方法进行获取，获取完后需要执行TypedArray中的recycle()方法。
