# Volley的简单使用

```
以StringRequest为例：

1.构建一个请求队列
  RequestQueue rq = new Volley().newRequestQueue(mContext);
  
2.构建一个请求体
  StringRequest sr = new StringRequest(Request.Method.GET, url, new       Response.Listener<String>() {
        @Override
        public void onResponse(String response) {

        }
    }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {

        }
    });
3.将请求体放入请求队列
    rq.add(sr);
```

# 目录：

- [Volley类解读](http://note.youdao.com/noteshare?id=e5ff49fb56ca9c794b212fc6fe4ed327&sub=8FA719C70DD5488080A4E308DD1175EA)

- [Request类解读](http://note.youdao.com/noteshare?id=e5ff49fb56ca9c794b212fc6fe4ed327&sub=8FA719C70DD5488080A4E308DD1175EA)

- [RequestQueue类解读](http://note.youdao.com/noteshare?id=45ac9e5317aba93d3b02adcf1e400a5a&sub=BD4CA64781494276A6E637A02FBF5AC9)

- [Request的子类和实现类解读](http://note.youdao.com/noteshare?id=167581bc37463118b683fb68aee18163&sub=30EEA4D8944F4FD883D7CCDBDA1D87A7)
> JsonRequest/JsonObjectReques/JsonArrayRequest/ImageRequest/StringRequest

- [ByteArrayPool类解读](http://note.youdao.com/noteshare?id=fd6a484d28fae0b175c9fa323b5930bd&sub=B45AC67BF7604079986616594F2F3A07)

- [PoolingByteArrayOutputStream类解读](http://note.youdao.com/noteshare?id=a9bc9b5f8879fe0c7c5a1f6d5e212eda&sub=434590D73D3F47B485314437982FA41D)

- [NetworkDispatcher和CacheDispatcher类解读](http://note.youdao.com/noteshare?id=a7c4f5170fb2f2e1a0ddb38e7fde3940&sub=C42E2D796DEA450187F1A2F102FB6744) 

- [RetryPolicy类解读](http://note.youdao.com/noteshare?id=d990d2dda4d329affed0130c3ae72d2d&sub=EE8DDCD73A6D4C1095416AB624B27040) 

- [Cache及其子类解读](http://note.youdao.com/noteshare?id=97909ce0b63f7edb4864ca57c65be5c4&sub=B5494AC2B04A4B2EB6F1ED2E0F468604) 



