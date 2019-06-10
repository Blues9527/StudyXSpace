**ImageCache**是ImageLoader类的一个内部接口类，用于实现图片缓存用的

```
public interface ImageCache {

    Bitmap getBitmap(String url);
    
    void putBitmap(String url, Bitmap bitmap);
}
```

**ImageListener**是ImageLoader的一个内部接口类


```
public interface ImageListener extends ErrorListener {

    void onResponse(ImageContainer response, boolean isImmediate);
}
```

**ImageContainer**类是ImageLoader的一个内部类

```
private Bitmap mBitmap;

private final ImageListener mListener;

private final String mCacheKey;

private final String mRequestUrl;

public ImageContainer(
        Bitmap bitmap, String requestUrl, String cacheKey, ImageListener listener) {
    mBitmap = bitmap;
    mRequestUrl = requestUrl;
    mCacheKey = cacheKey;
    mListener = listener;
}
```
```
//作用于主线程
@MainThread
public void cancelRequest() {
    //这里主要做一个线程判断，不是主线程则抛出异常
    Threads.throwIfNotOnMainThread();
    
    //没有监听，无法取消请求
    if (mListener == null) {
        return;
    }
    //根据缓存键去获取一个BatchedImageRequest实例
    BatchedImageRequest request = mInFlightRequests.get(mCacheKey);
    
    if (request != null) {
        //调用BatchedImageRequest里的removeContainerAndCancelIfNecessary方法
        boolean canceled = request.removeContainerAndCancelIfNecessary(this
        if (canceled) {
        //取消后移除缓存键
            mInFlightRequests.remove(mCacheKey);
        }
    } else {
        // check to see if it is already batched for delivery.
        request = mBatchedResponses.get(mCacheKey);
        if (request != null) {
            request.removeContainerAndCancelIfNecessary(this);
            if (request.mContainers.size() == 0) {
                mBatchedResponses.remove(mCacheKey);
            }
        }
    }
}
```

```
public Bitmap getBitmap() {
    return mBitmap;
}

public String getRequestUrl() {
    return mRequestUrl;
}
```

**BatchedImageRequest**类是ImageLoader的一个静态内部类，包装类用于映射请求

```
private final Request<?> mRequest;

private Bitmap mResponseBitmap;

private VolleyError mError;

//用ArrayList去存储数据
private final List<ImageContainer> mContainers = new ArrayList<>();

public BatchedImageRequest(Request<?> request, ImageContainer container) {
    mRequest = request;
    mContainers.add(container);
}

public void setError(VolleyError error) {
    mError = error;
}

public VolleyError getError() {
    return mError;
}

public void addContainer(ImageContainer container) {
    mContainers.add(container);
}

public boolean removeContainerAndCancelIfNecessary(ImageContainer container) {
    mContainers.remove(container);
    if (mContainers.size() == 0) {
    //如果列表里没有了imagecontainer，则取消请求
        mRequest.cancel();
        return true;
    }
    return false;
}
```

**ImageLoader**
```
//请求队列，用于存放请求
private final RequestQueue mRequestQueue;

//在第一个响应到达后所有响应等待的时间
private int mBatchResponseDelayMs = 100;

//L1缓存
private final ImageCache mCache;

private final HashMap<String, BatchedImageRequest> mInFlightRequests = new HashMap<>();

private final HashMap<String, BatchedImageRequest> mBatchedResponses = new HashMap<>();

//主线程handler
private final Handler mHandler = new Handler(Looper.getMainLooper());

private Runnable mRunnable;

public ImageLoader(RequestQueue queue, ImageCache imageCache) {
    mRequestQueue = queue;
    mCache = imageCache;
}
```

```
public static ImageListener getImageListener(
        final ImageView view, final int defaultImageResId, final int errorImageResId) {
    return new ImageListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            if (errorImageResId != 0) {
                view.setImageResource(errorImageResId);
            }
        }
        @Override
        public void onResponse(ImageContainer response, boolean isImmediate) {
            if (response.getBitmap() != null) {
                view.setImageBitmap(response.getBitmap());
            } else if (defaultImageResId != 0) {
                view.setImageResource(defaultImageResId);
            }
        }
    };
}
```

```
//判断是否能缓存，指定缩放类型为ScaleType.CENTER_INSIDE
public boolean isCached(String requestUrl, int maxWidth, int maxHeight) {
    return isCached(requestUrl, maxWidth, maxHeight, ScaleType.CENTER_INSIDE);
}

//只能作用于主线
@MainThread
public boolean isCached(String requestUrl, int maxWidth, int maxHeight, ScaleType scaleType) {
    //非主线抛出异常
    Threads.throwIfNotOnMainThread();
    //生成一个特殊的缓存key
    String cacheKey = getCacheKey(requestUrl, maxWidth, maxHeight, scaleType);
    return mCache.getBitmap(cacheKey) != null;
}
```

```
//从请求url返回一个imagecontainer，默认宽高，默认ScaleType为ScaleType.CENTER_INSIDE
public ImageContainer get(String requestUrl, final ImageListener listener) {
    return get(requestUrl, listener, /* maxWidth= */ 0, /* maxHeight= */ 0);
}

//限定ScaleType为ScaleType.CENTER_INSIDE
public ImageContainer get(
        String requestUrl, ImageListener imageListener, int maxWidth, int maxHeight) {
    return get(requestUrl, imageListener, maxWidth, maxHeight, ScaleType.CENTER_INSIDE);
}

//只能在主线调用
@MainThread
public ImageContainer get(
        String requestUrl,
        ImageListener imageListener,
        int maxWidth,
        int maxHeight,
        ScaleType scaleType) {
    //非主线，抛出异常
    Threads.throwIfNotOnMainThread();
    
    //获取特殊的缓存key
    final String cacheKey = getCacheKey(requestUrl, maxWidth, maxHeight, scaleType);
    
    //根据缓存key去获取bitmap
    Bitmap cachedBitmap = mCache.getBitmap(cacheKey);
    if (cachedBitmap != null) {
    //bitmap不为空则实例化一个imagecontainer
        ImageContainer container =
                new ImageContainer(
                        cachedBitmap, requestUrl, null, null);
        //将imagecontainer传递出去
        imageListener.onResponse(container, true);
        return container;
    }
    //如果bitmap不存在，则获取一个
    ImageContainer imageContainer =
            new ImageContainer(null, requestUrl, cacheKey, imageListener);
    imageListener.onResponse(imageContainer, true);
    
    //构建一个批处理请求
    BatchedImageRequest request = mInFlightRequests.get(cacheKey);
    if (request == null) {
        request = mBatchedResponses.get(cacheKey);
    }
    if (request != null) {
        //将请求添加到批处理请求 
        request.addContainer(imageContainer);
        return imageContainer;
    }
    
    //构建一个真正的请求
    Request<Bitmap> newRequest =
            makeImageRequest(requestUrl, maxWidth, maxHeight, scaleType, cacheKey);
    //将请求添加至请求队列
    mRequestQueue.add(newRequest);
    mInFlightRequests.put(cacheKey, new BatchedImageRequest(newRequest, imageContainer));
    return imageContainer;
}

//实例化一个ImageRequest返回一个 Request<Bitmap>
protected Request<Bitmap> makeImageRequest(
        String requestUrl,
        int maxWidth,
        int maxHeight,
        ScaleType scaleType,
        final String cacheKey) {
    return new ImageRequest(
            requestUrl,
            new Listener<Bitmap>() {
                @Override
                public void onResponse(Bitmap response) {
                    onGetImageSuccess(cacheKey, response);
                }
            },
            maxWidth,
            maxHeight,
            scaleType,
            Config.RGB_565,
            new ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    onGetImageError(cacheKey, error);
                }
            });
}

protected void onGetImageSuccess(String cacheKey, Bitmap response) {
    
    //将bitmap放入缓存
    mCache.putBitmap(cacheKey, response);
    
    //拿到一个移除指定缓存key后的BatchedImageRequest
    BatchedImageRequest request = mInFlightRequests.remove(cacheKey);
    if (request != null) {
        //对bitmap赋值
        request.mResponseBitmap = response;
        batchResponse(cacheKey, request);
    }
}


protected void onGetImageError(String cacheKey, VolleyError error) {

    //获取图片失败后，拿到一个移除指定缓存key后的BatchedImageRequest
    BatchedImageRequest request = mInFlightRequests.remove(cacheKey);
    if (request != null) {
        //设置error
        request.setError(error);
        batchResponse(cacheKey, request);
    }
}

//批处理响应
private void batchResponse(String cacheKey, BatchedImageRequest request) {
    //放入响应待处理hashmap
    mBatchedResponses.put(cacheKey, request);
    if (mRunnable == null) {
        //初始runnable为空，实例化一个
        mRunnable = new Runnable() {
                    @Override
                    public void run() {
                        //遍历批处理图片请求
                        for (BatchedImageRequest bir : mBatchedResponses.values()) {
                             //遍历存放图片容器的ArrayList
                            for (ImageContainer container : bir.mContainers) {
                                //监听为空，continue
                                if (container.mListener == null) {
                                    continue;
                                }
                                //如果volley error不为空，将container传递出去
                                if (bir.getError() == null) {
                                    container.mBitmap = bir.mResponseBitmap;
                                    container.mListener.onResponse(container, false);
                                } else {
                                   //将错误传递出去 container.mListener.onErrorResponse(bir.getError());
                                }
                            }
                        }
                        //处理完后清空掉hashmap
                        mBatchedResponses.clear();
                        //runnable置空
                        mRunnable = null;
                    }
                };
        //handler延时处理runnable
        mHandler.postDelayed(mRunnable, mBatchResponseDelayMs);
    }
}

//创建一个用于L1缓存的缓存键
private static String getCacheKey(
        String url, int maxWidth, int maxHeight, ScaleType scaleType) {
    return new StringBuilder(url.length() + 12)
            .append("#W")
            .append(maxWidth)
            .append("#H")
            .append(maxHeight)
            .append("#S")
            .append(scaleType.ordinal())
            .append(url)
            .toString();
}
```








