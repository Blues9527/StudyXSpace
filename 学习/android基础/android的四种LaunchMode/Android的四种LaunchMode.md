# activity的四种启动模式
    1)standard 每次启动都new 一个instance（使用场景，程序入口）
    2)singleTask 
    判断activity栈是否有一个instance，有就移除target之前的将自己变成栈顶instance（适用场景，程序入口启动页），没有则在栈顶new一个
    3)singleTop
    判断栈顶是否有一个instance，有就复用，没有就new（适用场景，消息推送跳转） 
    4)singleInstance 开辟私有的task，完全独立于程序的其他activity的task(独立页面)
    
    
# 综合分析

> Activity: 共有3个，Activity1 、 Activity2 、Activity3

> 固定的跳转顺序：App启动 1 1 2 3 1 2 返回

### standard模式

执行的顺序:
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 3 2 1 1 1 退出app

分析：
> Activity使用standard模式进行跳转的时候，每次跳转到新页面（包括自身跳转到自身和自身跳转到其他）都会新启动一个页面，每次都会调用新Activity的onCreate()方法。


### singleTop模式

执行的顺序：
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 3 2 1 退出app

分析：
> Activity使用singleTop模式进行跳转的时候，跳转至新的页面时，如果新页面在activity栈顶的话，则复用，就好像1 跳转至 1再跳转至 1，因为App启动的时候栈顶activity就是Activity1，所以 1→1→1 时只有首次启动时Activity1才调用了onCreate()方法，后续的两次自身跳转到自身并没有调用onCreate()方法。而后面的由 3→1时，栈顶activity是Activity3并不是Activity1，所以又会调用Activity1的onCreate()方法。

> 由返回的顺序看，很明显Activity1在栈顶时，自身跳转到自身并没有实例化一个新的activity，所以由 2→1后就直接退出app了。

### singleTask模式

执行的顺序：
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 退出App

分析：
> 首先由启动App(1) → 1 → 1，因为Activity栈村咋子Activity1的实例，则复用Activity1的实例，然后跳转到Activity2Activity3都是正常跳转，但是再由Activity3跳转回Activity1时，由于Activity栈中已有Activity1的实例，而Activity1并不处于栈顶，所以移除Activity2Activity3，即Activity2Activity3都调用了onDestroy()方法，此时Activity栈只有Activity1一个实例，再跳转Activity2时，则正常实例化一个Activity2。

> 返回时，由Activity2回到Activity1，此时，Activity栈已吗，没有其他实例了，所以再按返回键App退出。

### singleInstance模式

执行的顺序：
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 3 退出App

分析：
> 单例顾名思义就是Activity栈里面只能有Activity1/2/3的一个实例，如果Activity里面没有，就实例化一个Activity并入栈，有的话就复用并置于栈顶。

> 启动App时生成了Activity1的实例，所以再跳转至Activity1是不会重新创建实例的，然后跳转Activity2Activity3时，Activity栈没有实例则都创建一个，由3→1时，由于Activity栈已经有Activity1的实例了，所以复用Activity1并置于栈顶，1→2也是同理。

> 点击返回就是出栈顺序了，跳转结束后栈的顺序是 2→1→3（栈顶开始），所以2先出栈，再到1，再到3。
