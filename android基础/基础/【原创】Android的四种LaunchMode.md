# 综合分析

> Activity: 共有3个，Activity1 、 Activity2 、Activity3

> 固定的跳转顺序：App启动(1) 1 1 2 3 1 2 返回

拓展：
> 栈：是一种数据结构，遵循先进后出（后进先出）的规则，常用FILO或者LIFO表示。

> 队列：是一种数据结构，遵循先进先出的规则，常用FIFO表示。

### standard模式

执行的顺序:
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 3 2 1 1 1 退出app

分析：
> Activity使用standard模式进行跳转的时候，每次跳转到新页面（包括自身跳转到自身和自身跳转到其他）都会新启动一个页面，每次都会调用新Activity的onCreate()方法。

总结：
> 当activity使用standard模式的时候，每次跳转都new一个activity实例，activity跳转顺序也是activity进栈顺序。每次返回都是一个activity出栈操作。


### singleTop模式

执行的顺序：
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 3 2 1 退出app

分析：
> Activity使用singleTop模式进行跳转的时候，跳转至新的页面时，如果新页面在activity栈顶的话，则复用，就好像1 跳转至 1再跳转至 1，因为App启动的时候栈顶activity就是Activity1，所以 1→1→1 时只有首次启动时Activity1才调用了onCreate()方法，后续的两次自身跳转到自身并没有调用onCreate()方法。而后面的由 3→1时，栈顶activity是Activity3并不是Activity1，所以又会调用Activity1的onCreate()方法。

> 由返回的顺序看，很明显Activity1在栈顶时，自身跳转到自身并没有实例化一个新的activity，所以由 2→1后就直接退出app了。

总结：
> 当activity使用singleTop模式的时候，会先判断栈顶是否有对应activity的实例，如果有就复用，没有就new一个activity的实例置于栈顶。对比standard模式，最大的区别就是：如果目标activity处于栈顶时，自身跳转自身并不会生成多个实例activity。

### singleTask模式

执行的顺序：
> 跳转顺序 App启动(1) 1 1 2 3 1 2

> 返回顺序 1 退出App

分析：
> 首先由启动App(1) → 1 → 1，因为Activity栈村咋子Activity1的实例，则复用Activity1的实例，然后跳转到Activity2Activity3都是正常跳转，但是再由Activity3跳转回Activity1时，由于Activity栈中已有Activity1的实例，而Activity1并不处于栈顶，所以移除Activity2Activity3，即Activity2Activity3都调用了onDestroy()方法，此时Activity栈只有Activity1一个实例，再跳转Activity2时，则正常实例化一个Activity2。

> 返回时，由Activity2回到Activity1，此时，Activity栈已吗，没有其他实例了，所以再按返回键App退出。

用途：
> 登录注册框

总结：
> 当activity使用singleTask模式时，首先会判断activity栈里面是否有目标activity的实例，如果有就再判断目标activity的实例位于栈的哪个位置，如果位于栈顶，就复用目标activity的实例，如果不是位于栈顶，就移除目标activity前所有其他activity的实例，使目标activity自身置于栈顶。如果activity栈没有目标activity的实例，就new一个实例置于栈顶。


### singleInstance模式

参与的类：
> Activity1 2 3 4

执行的顺序：
> 跳转顺序 App启动(1)  1 2 3 4  

> 返回顺序 3 2 1 退出app

分析：
> Activity 默认有3个任务栈，未使用的Activity任务栈的id都为-1，首次启动App的时候，无论是什么模式都会在第一个Activity任务栈中实例化一个对象，如果要跳转的对象Activity的启动模式时singleInstance时，则会在另外一个Activity任务栈中实例化一个对象。当使用singleInstance的activity超过3个时，每次跳转到新的singleInstance的activity时就会新建一个activity任务栈。

用途：
> 微信聊天

总结：
> 可以理解为，使用singleInstance时，就是一个萝卜一个坑。一个singleInstance模式的Activity对应一个Activity任务栈。


# activity的四种启动模式简单总结

> 1.standard 每次启动都new 一个instance，无论Activity里之前是否有实例的存在。

> 2.singleTask 先判断栈顶有没有该Activity的实例，如果有则复用，如果该Activity的实例不在栈顶，则移除该Activity前所有的对象，使其自己成为栈顶。如果没有实例，则在栈顶new一个实例。Activity可能又多个实例。

> 3.singleTop 如果栈顶里面有，就复用，没有就new一个。

> 4.singleInstance 会开辟一个独立的栈。

# 启动模式之间的区别

> singleInstance与其他3中启动模式的区别就是：会有多个Activity任务栈在使用，而其他三种启动模式都只有一个Activity栈。

> singleTask和singleInstance是保证了Activity栈只有一个目标Activity的实例，而standard和singleTop不能保证Activity是唯一的实例。

### 打印任务栈的代码
```
  List<ActivityManager.AppTask> appTasks = ((ActivityManager) getSystemService(ACTIVITY_SERVICE)).getAppTasks();

        for (ActivityManager.AppTask appTask : appTasks) {
            Log.i("Blues", "activity_id ---> " + appTask.getTaskInfo().id);
            Log.i("Blues", "top_activity ---> " + appTask.getTaskInfo().topActivity);
        }
```
