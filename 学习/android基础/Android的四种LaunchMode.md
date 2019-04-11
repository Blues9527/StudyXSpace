# activity的四种启动模式
    1)standard 每次启动都new 一个instance（使用场景，程序入口）
    2)singleTask 
    判断activity栈是否有一个instance，有就移除target之前的将自己变成栈顶instance（适用场景，程序入口启动页），没有则在栈顶new一个
    3)singleTop
    判断栈顶是否有一个instance，有就复用，没有就new（适用场景，消息推送跳转） 
    4)singleInstance 开辟私有的task，完全独立于程序的其他activity的task(独立页面)