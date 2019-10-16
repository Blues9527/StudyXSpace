# 6.Android中常用的网络请求框架的简单使用
### OkHttp
功能强大，可封装
    
```
使用OkHttp或者Retrofit时可以添加OkHttp拦截器进行网络url和数据的打印

//建造者模式 
    OkHttpClient client = new OkHttpClient.Builder()
                            .addInterceptor(Interceptor)//添加云端拦截器
                            .connectTimeout(Time)//连接超时时间
                            .readTimeout(Time)//读取超时时间
                            .writerTimeout(Time)//写入超时时间
                            .build();
```

    
```
    //建造者模式  
    1.构建一个请求
    Request request - new Request.Builder()
                        .url(Url)
                        .get()//使用get方法进行请求
                        .post(RequestBody)//使用post方法进行请求,传入一个请求体
                        .build();
    2.将request放入队列
    Call call = new OkHttpClient().newCall(request);
    
    3.使用异步进行请求
    call.enqueue(new CallBack(){
        void onFailure(Call call,IOException e);
        void onSuccess(Call call,Respose response);
    });
    
    4.使用同步请求
    call.execute();
    
    5.解析异步返回的数据
    常用的是gson解析
    
```
    
### Retrofit
    是基于okhttp的封装的网络请求框架
    
```
    1.构建Retrofit.Builder()
    Retrofit retrofit = new Retrofit.Builder()
                            .baseUrl(String url)//必须以"/"结尾
                            .client(OkHttpClient)//使用OkHttp进行网络请求
                             .addConverterFactory(GsonConverterFactory.create())//使用gson解析数据
                             .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//使用RxJava进行数据装载
                            .build();
    
    2.实例化API类
    API api = retrofit.create(API.class);
    
    3.进行网络请求
    @JavaBean 需要映射的实体类
    @getCall 接口类中的方法
    @Params 方法中需要传入的参数
    通过注解去标注方法的请求方式
    常用的方法有@GET("extraUrl")/@POST("extraUrl")
    Call<JavaBean> call = api.getCall(Params);
    
    //此处基本与OkHttp相同，但是不同的地方在于callback类中带有泛型实体类
    call.enqueue(new CallBack<JavaBean>(){
        void onFailure(Call<JavaBean> call,Throwable t);
        void onSuccess(Call<JavaBean> call,Response<JavaBean> reponse);
    });
```

### Volley
    为google官方的网络请求框架
    
```
1.构建一个请求队列
RequestQueue rq = new Volley.newRequestQueue(Context);

2.构建一个请求体
@method 请求方法
@url 请求url
@listener 成功监听
@errorListener 失败监听
StringRequest sr = new StringRequest(int method, String url, Listener<T> listener, ErrorListener errorListener);

3.将请求体放入请求队列
rq.add(sr);
```


### 原生HttpUrlConnection

```
    HttpURLConnection connection = (HttpURLConnection)new URL(url).openConnection();//开启连接
    connection.setConnectionTimeout(Time);//设置连接超时时间
    connection.setReadTimeout(Time);//设置读取超时时间
    connection.connect();
```