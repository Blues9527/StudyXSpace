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