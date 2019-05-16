
1.通常函数，类都需要传入self，使用时报缺少参数self标识函数或者类没有初始化。

```
ex:

class DBHelper:
    def __init__(self):
        ...
    
    def connect():
        ...
        
使用时：
DBHelper.connect() 报缺少参数self

正确使用: DBHelper().connect()
```
