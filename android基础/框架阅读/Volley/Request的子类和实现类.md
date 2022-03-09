### Request类是一个抽象类，接下来看一下Request的子类和实现类
> Request的实现类有5个，分别是StringReqeust、JsonRequest（抽象类）、JsonObjectRequest、JsonArrayRequest、ImageRequest

**JsonRequest类**
> 继承自Request类，也是一个抽象类，同时也是JsonArrayRequest和JsonObjectRequest的父类。

> 注意一点，重写parseNetworkResponse方法也并没有具体实现，还是abstract。

> 对比Request，JsonRequest多传入了一个Response.Listener监听，在deliverResponse方法里将response回调出去。

```
//默认采用utf-8编码格式
protected static final String PROTOCOL_CHARSET = "utf-8";

//对象锁，保护mListener在调用cancel()的时候被清除，在传输的时候被读取。
private final Object mLock = new Object();

//传入默认的method Method.DEPRECATED_GET_OR_POST
@Deprecated
JsonRequest(
            String url, 
            String requestBody, 
            Listener<T> listener, 
            ErrorListener errorListener){
        this(Method.DEPRECATED_GET_OR_POST, url, requestBody, listener, errorListener);
    }

JsonRequest(
            int method,
            String url,
            @Nullable String requestBody,
            Listener<T> listener,
            @Nullable ErrorListener errorListener){
        super(method, url, errorListener);
        mListener = listener;
        mRequestBody = requestBody;
    }
  
//调用cancel的时候，清除mListener          
@Override
    public void cancel() {
        super.cancel();
        synchronized (mLock) {
            mListener = null;
        }
    }
    
@Override
protected void deliverResponse(T response) {
    Response.Listener<T> listener;
    synchronized (mLock) {
        //同步mListener
        listener = mListener;
    }
    if (listener != null) {
        //listener不为空的时候response出去
        listener.onResponse(response);
    }
}

//没有具体去实现这个方法，还是交由子类去具体实现
@Override
protected abstract Response<T> parseNetworkResponse(NetworkResponse response);
            
```

JsonObjectRequest类

```
//
JsonObjectRequest(
            int method,
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener){
        super(
                method,
                url,
                (jsonRequest == null) ? null : jsonRequest.toString(),
                listener,
                errorListener);
    }

//根据传进来的jsonRequest是否为空来判断是GET请求还是POST请求         
JsonObjectRequest(
            String url,
            @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener,
            @Nullable ErrorListener errorListener){
        this(
                jsonRequest == null ? Method.GET : Method.POST,
                url,
                jsonRequest,
                listener,
                errorListener);
    }
    

    @Override
    protected Response<JSONObject> parseNetworkResponse(NetworkResponse response) {
        try {
            String jsonString =
                    new String(
                            response.data, HttpHeaderParser.parseCharset(response.headers, PROTOCOL_CHARSET));
            return Response.success(
            //通过JSONObject的方式将数据传递出去
                    new JSONObject(jsonString), HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JSONException je) {
            return Response.error(new ParseError(je));
        }
    }

```

**JsonArrayRequest类**

```
//调用父类方法，传入默认method为Method.GET，requestBody为null
 JsonArrayRequest(
            String url, 
            Listener<JSONArray> listener, 
            @Nullable ErrorListener errorListener){
        super(Method.GET, url, null, listener, errorListener);
    }

//调用父类方法，传入requestBody (jsonRequest == null) ? null : jsonRequest.toString()            
JsonArrayRequest(
            int method,
            String url,
            @Nullable JSONArray jsonRequest,
            Listener<JSONArray> listener,
            @Nullable ErrorListener errorListener){
        super(
                method,
                url,
                (jsonRequest == null) ? null : jsonRequest.toString(),
                listener,
                errorListener);
    }
    
@Override
    protected Response<JSONArray> parseNetworkResponse(NetworkResponse response) {
        try {
            String jsonString =
                    new String(
                            response.data, HttpHeaderParser.parseCharset(response.headers, PROTOCOL_CHARSET));
            return Response.success(
            //通过JSONArray的方式将数据传递出去
                    new JSONArray(jsonString), HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JSONException je) {
            return Response.error(new ParseError(je));
        }
    }

```

**ImageRequest类**

```
//默认超时时间为 1s
public static final int DEFAULT_IMAGE_TIMEOUT_MS = 1000;
```

```
//默认最大重试次数为2
public static final int DEFAULT_IMAGE_MAX_RETRIES = 2;
```

```
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            ScaleType scaleType,
            Config decodeConfig,
            @Nullable Response.ErrorListener errorListener){
            //默认传入method为Method.GET
        super(Method.GET, url, errorListener);
        //设置重试策略
        setRetryPolicy(
        //实例化一个默认的重试策略，传入设定的值
                new DefaultRetryPolicy(
                        DEFAULT_IMAGE_TIMEOUT_MS,//1s
                        DEFAULT_IMAGE_MAX_RETRIES,//2次
                        DEFAULT_IMAGE_BACKOFF_MULT));//2f
        mListener = listener;
        mDecodeConfig = decodeConfig;
        mMaxWidth = maxWidth;
        mMaxHeight = maxHeight;
        mScaleType = scaleType;
    }


@Deprecated
ImageRequest(
            String url,
            Response.Listener<Bitmap> listener,
            int maxWidth,
            int maxHeight,
            Config decodeConfig,
            Response.ErrorListener errorListener){
        this(
                url,
                listener,
                maxWidth,
                maxHeight,
                ScaleType.CENTER_INSIDE,//默认传入ScaleType.CENTER_INSIDE
                decodeConfig,
                errorListener);
    }

@Override
    public Priority getPriority() {
        return Priority.LOW;//默认优先级最低
    }

//缩放矩形的一边以适应长宽比
private static int getResizedDimension(
            int maxPrimary,
            int maxSecondary,
            int actualPrimary,
            int actualSecondary,
            ScaleType scaleType) {

        //如果根本没有优势值，就返回实际值
        if ((maxPrimary == 0) && (maxSecondary == 0)) {
            return actualPrimary;
        }
    
        //如果ScaleType是FIT_XY，也就是填充满整个矩形，就忽略比例
        if (scaleType == ScaleType.FIT_XY) {
            if (maxPrimary == 0) {
                return actualPrimary;
            }
            return maxPrimary;
        }

        //如果主节点未指定，则缩放主节点以匹配辅助节点的缩放比例
        if (maxPrimary == 0) {
            double ratio = (double) maxSecondary / (double) actualSecondary;
            return (int) (actualPrimary * ratio);
        }

        //
        if (maxSecondary == 0) {
            return maxPrimary;
        }

        double ratio = (double) actualSecondary / (double) actualPrimary;
        int resized = maxPrimary;

        //如果是ScaleType.CENTER_CROP类型的话，填充整个矩形，保持宽高比
        if (scaleType == ScaleType.CENTER_CROP) {
            if ((resized * ratio) < maxSecondary) {
                resized = (int) (maxSecondary / ratio);
            }
            return resized;
        }

        if ((resized * ratio) > maxSecondary) {
            resized = (int) (maxSecondary / ratio);
        }
        return resized;
    }
    
 @Override
    protected Response<Bitmap> parseNetworkResponse(NetworkResponse response) {
        synchronized (sDecodeLock) {
            try {
                return doParse(response);
            } catch (OutOfMemoryError e) {
                VolleyLog.e("Caught OOM for %d byte image, url=%s", response.data.length, getUrl());
                return Response.error(new ParseError(e));
            }
        }
    }
            
//核心方法，调用此方法解析出来一个bitmap并通过Response.success/error将结果传递出去。
 private Response<Bitmap> doParse(NetworkResponse response) {
        byte[] data = response.data;
        //实例化一个bitmap解码设置
        BitmapFactory.Options decodeOptions = new BitmapFactory.Options();
        Bitmap bitmap = null;
        
        //如果传进来的款到都是0，则采用默认Bitmap.Config.ARGB_8888
        if (mMaxWidth == 0 && mMaxHeight == 0) {
            decodeOptions.inPreferredConfig = mDecodeConfig;
            bitmap = BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
        } else {
        //如果要调整图像的大小，首先要得到真实的的宽高
            decodeOptions.inJustDecodeBounds = true;
            BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
            int actualWidth = decodeOptions.outWidth;
            int actualHeight = decodeOptions.outHeight;

            //然后计算我们想要解码的图片大小
            int desiredWidth =
                    getResizedDimension(
                            mMaxWidth, mMaxHeight, actualWidth, actualHeight, mScaleType);
            int desiredHeight =
                    getResizedDimension(
                            mMaxHeight, mMaxWidth, actualHeight, actualWidth, mScaleType);

            decodeOptions.inJustDecodeBounds = false;
            decodeOptions.inSampleSize =
                    findBestSampleSize(actualWidth, actualHeight, desiredWidth, desiredHeight);
            Bitmap tempBitmap = BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);

        //如果解码得到一个bitmap不为空，然后得到的bitmap的宽或高任一大于其理想的宽高，则调用Bitmap的createScaledBitmap方法去获取一个bitmap的一个实例。使用完后进行bitmap的回收
            if (tempBitmap != null
                    && (tempBitmap.getWidth() > desiredWidth
                            || tempBitmap.getHeight() > desiredHeight)) {
                bitmap = Bitmap.createScaledBitmap(tempBitmap, desiredWidth, desiredHeight, true);
                tempBitmap.recycle();
            } else {
            //反之，符合条件则直接将tempBitmap赋值给bitmap
                bitmap = tempBitmap;
            }
        }

        if (bitmap == null) {
            return Response.error(new ParseError(response));
        } else {
            return Response.success(bitmap, HttpHeaderParser.parseCacheHeaders(response));
        }
    }
    
    //清除mListener
     @Override
    public void cancel() {
        super.cancel();
        synchronized (mLock) {
            mListener = null;
        }
    }
    
    //先同步mListener，然后通过接口response出去
    @Override
    protected void deliverResponse(Bitmap response) {
        Response.Listener<Bitmap> listener;
        synchronized (mLock) {
            listener = mListener;
        }
        if (listener != null) {
            listener.onResponse(response);
        }
    }
    
    
    //返回用于向下缩放位图的最大二乘方数，该位图不会导致缩放超过所需大小
    @VisibleForTesting
    static int findBestSampleSize(
            int actualWidth, int actualHeight, int desiredWidth, int desiredHeight) {
        double wr = (double) actualWidth / desiredWidth;
        double hr = (double) actualHeight / desiredHeight;
        double ratio = Math.min(wr, hr);
        float n = 1.0f;
        while ((n * 2) <= ratio) {
            n *= 2;
        }

        return (int) n;
    }
```

**StringRequest类**

```
//调用父类方法，传入限定String型的Listener
StringRequest(
            int method,
            String url,
            Listener<String> listener,
            @Nullable ErrorListener errorListener){
        super(method, url, errorListener);
        mListener = listener;
    }

//调用上面方法，传入默认的method Method.GET     
StringRequest(
            String url, Listener<String> listener, @Nullable ErrorListener errorListener){
        this(Method.GET, url, listener, errorListener);
    }


//同步mListener,并置空
@Override
    public void cancel() {
        super.cancel();
        synchronized (mLock) {
            mListener = null;
        }
    }

//同步mListener，然后通过接口将response传递出去   
@Override
    protected void deliverResponse(String response) {
        Response.Listener<String> listener;
        synchronized (mLock) {
            listener = mListener;
        }
        if (listener != null) {
            listener.onResponse(response);
        }
    }
    
@Override
    @SuppressWarnings("DefaultCharset")
    protected Response<String> parseNetworkResponse(NetworkResponse response) {
        String parsed;
        try {
            parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
        } catch (UnsupportedEncodingException e) {
            parsed = new String(response.data);
        }
        return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
    }
```