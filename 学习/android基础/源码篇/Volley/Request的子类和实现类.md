### Request类是一个抽象类，接下来看一下Request的子类和实现类
> Request的实现类有5个，分别是StringReqeust、JsonRequest（抽象类）、JsonObjectRequest、JsonArrayRequest、ImageRequest

JsonRequest类
> 继承自Request类，也是一个抽象类，同时也是JsonArrayRequest和JsonObjectRequest的父类。

> 注意一点，重写parseNetworkResponse方法也并没有具体实现，还是abstract。

> 对比Request，JsonRequest多传入了一个Response.Listener监听，在deliverResponse方法里将response回调出去。

```
@Deprecated
JsonRequest(
            String url, 
            String requestBody, 
            Listener<T> listener, 
            ErrorListener errorListener)

JsonRequest(
            int method,
            String url,
            @Nullable String requestBody,
            Listener<T> listener,
            @Nullable ErrorListener errorListener)
            
上面的方法调用了下方的方法，传入method参数Method.DEPRECATED_GET_OR_POST
```

JsonObjectRequest类

```
JsonObjectRequest(
            int method,
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener)
            
JsonObjectRequest(
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener)
下面的方法调用了上面的方法，传进去method jsonRequest == null ? Method.GET : Method.POST

最终都是调用父类的方法，只不过限定了Listener的传入类型为JSONObject

```

JsonArrayRequest类

```
 JsonArrayRequest(
            String url, 
            Listener<JSONArray> listener, 
            @Nullable ErrorListener errorListener)
调用父类方法，传入默认method为Method.GET，requestBody为null
            
JsonArrayRequest(
            int method,
            String url,
            @Nullable JSONArray jsonRequest,
            Listener<JSONArray> listener,
            @Nullable ErrorListener errorListener)
调用父类方法，传入requestBody (jsonRequest == null) ? null : jsonRequest.toString()
```

ImageRequest类
> 默认超时时间为 1s

> 默认最大重试次数为2

> 限定传进来的Listener的类型为Bitmap

> 通过锁去限制每次解码不超过一张图片

```
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            ScaleType scaleType,
            Config decodeConfig,
            @Nullable Response.ErrorListener errorListener)
默认传入method为Method.GET

@Deprecated
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            Config decodeConfig,
            Response.ErrorListener errorListener)
调用上方方法，传入默认配置ScaleType.CENTER_INSIDE

getPriority()//return Priority.LOW

getResizedDimension(
            int maxPrimary,
            int maxSecondary,
            int actualPrimary,
            int actualSecondary,
            ScaleType scaleType)
            
doParse(NetworkResponse response)//核心方法，调用此方法解析出来一个bitmap并通过Response.success/error将结果传递出去。
```

StringRequest类

```
StringRequest(
            int method,
            String url,
            Listener<String> listener,
            @Nullable ErrorListener errorListener)
//调用父类犯法，传入限定String型的Listener
            
StringRequest(
            String url, Listener<String> listener, @Nullable ErrorListener errorListener)
//调用上面方法，传入默认的method Method.GET
```