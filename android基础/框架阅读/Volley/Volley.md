
> 1.Volley类主要是为了创建一个RequestQueue的实例

> 2.Volley类中所有的方法都是静态的，所以可以直接 new Volley().newRequestQueue(...)来实例化一个RequestQueue


```
//考虑到HttpStack时被弃用的状态，随时都会被移除出去，首先判断传进来的BaseHttpStack是否为空，如果为空则判断当前android sdk api版本，如果api>9则network = new BasicNetwork(new HurlStack());否则就 network =new BasicNetwork(new HttpClientStack(AndroidHttpClient.newInstance(userAgent)));如果stack不为空，则 network = new BasicNetwork(stack)
1.public static RequestQueue newRequestQueue(Context context, BaseHttpStack stack)

//先判断传进来的HttpStack是否为空，为空则调用4，4再调用1，1进入到stack==null的模块，最终是调用3；不为空的话则直接调用3
 @Deprecated
2.public static RequestQueue newRequestQueue(Context context, HttpStack stack)


//最终产生RequestQueue实例的一个方法。先生成一个Volley缓存的文件目录，然后new RequestQueue生成实例，最后调用RequestQueue的start方法开启缓存调度和网络调度。
3.private static RequestQueue newRequestQueue(Context context, Network network) 

//直接调用1，传个null的stack，接着再调用3
4.public static RequestQueue newRequestQueue(Context context)
```

分析：
> 上述方法中，只有3是private的，一般private方法都是最终方法的入口。
上述调用的顺序是：1->3 /2->3 or 2->4->1->3 /4->1->3


