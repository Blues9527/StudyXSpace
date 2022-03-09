[toc]
# View的绘制流程

## 绘制流程时序图
[时序图](https://www.processon.com/view/link/61b091371e085322dab98379)

## ViewRootImpl的创建
  1. 首先是`ActivityThread.handleLaunchActivity()`内部会执行`performLaunchActivity()`

  2. `performLaunchActivity()`内部会调用`activity.attach()`

  3. 在`attach()`内部会`new PhoneWindow()`和获取`WindowManager`并赋值给`phoneWindow`

  4. 然后是`performLaunchActivity()`内部将调用`Instrumentation.callActivityOnCreate()`

  5. `callActivityOnCreate()`内部会调用`activity.performCreate()`，内部继续调用`Activity.onCreate()`

  6. 此时会走到`setContentView()`，如果`activity`是继承自`AppcompactActivity.setContentView()`会走到`AppCompatDelegateImpl`的`setContentView()`，调用链 `ensureSubDecor() -> createSubDecor()-> mWindow.setContentView()`，接着就是`phoneWindow.setContentView(View view)`继续往下调用双参的`setContentView(View,LayoutParams)`。而直接继承`Activity`的话，就是直接调用`phoneWindow.setContentView(View view)`，然后继续往下走。

  7. 接着`PhoneWindow`里的`setContentView()`会调用`installDecor()`来创建`DecorView`，分别执行`generateDecor()`来创建`DecorView`和`generateLayout()`来获取布局id为`R.id.content`的`ViewGroup`。

  8. 然后继续走`ActivityThread.handleResumeActivity()`，此时内部会调用`WindowManger.addView()`方法将`DecorView`添加到`Window`里去。

  9. `WindowManager`有唯一的实现类是`WindowManagerImpl`，即会调用到`WindowManagerImpl.addView()`方法，其内部会调用`WindowManagerGlonbal.addView()`方法

 10. `WindowManagerGlonbal.addView()`内部会实例化一个`ViewRootImpl`。

## View的绘制流程
  1. `WindowManagerGlonbal`会接着调用`ViewRootImpl.setView()`，紧接着会调用`requestLayout()`

  2. `requestLayout()`内部会先设置`mLayoutRequested`为`true`，然后调用`scheduleTraversals()`

  3. `scheduleTraversals()`内部会先获取`MessageQueue`并`postSyncBarrier()`暂停同步消息的处理，然后调用`Choreographer.postCallback()`，传入`TraversalRunnable`并执行

  4. `TraversalRunnable`顾名思义是一个`Runnable`主要关注其`run`方法实现就是`doTraversal()`

  5. `doTraversal()`首先会移除同步消息屏障`removeSyncBarrier()`，然后调用`performTraversals()`

  6. `performTraversals()`就是绘制流程的核心方法了，会先后执行`performMeasure()`、`performLayout()`、`performDraw()`，其中会根据`mLayoutRequested`标识来判断是否执行`performMeasure()`、`performLayout()`，这里涉及到`view`的`requestLayout()`和`invalidate()`方法的区别
  
  7. `performMeasure()`会执行两次，第一次测量的是`Window`的大小，第二次才是测量控件树的大小，会调用`DecorView的measure()`方法
  
  8. `performLayout()`会调用到`DecorView`的`layout()`方法，执行完`performLayout()`后，会回调`mAttachInfo.mTreeObserver.dispatchOnGlobalLayout()`，然后可以监听这个方法拿到`View的`宽高
  
  9. `performDraw()`内部会调用`draw(boolean fullRedrawNeeded)`，然后会判断是否开启了硬件加速`if (mAttachInfo.mThreadedRenderer != null && mAttachInfo.mThreadedRenderer.isEnabled())`，如果开启了就是使用硬件绘制`mAttachInfo.mThreadedRenderer.draw()`，否则就使用软件绘制`drawSoftware()`，到`DecorView.draw()`方法，其中`draw`的顺序流程是
     - 1. `drawBackground(canvas)`：绘制背景
     - 2. 如果需要，则保存`canvas`的图层以备褪色
     - 3. `onDraw(canvas)`：绘制`view`的内容 
     - 4. `dispatchDraw(canvas)`：绘制view的子视图
     - 5. 如果需要，则绘制褪色区域并恢复图层
     - 6. `onDrawForeground(canvas)`：绘制装饰（如滚动条）
     - 7. `drawDefaultFocusHighlight(canvas)`：绘制默认焦点的高光

### 1.绘制前准备内容

```Java
//1.1首先执行handleLaunchActivity，会创建一个Activity然后执行Activity的attach()
//attach方法会创建phoneWindow和windowManager的实例
attach(){
//忽略部分代码...

    //创建PhoneWindow
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    
    //设置WindowManager
    mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
    
    //赋值WindowManager
    mWindowManager = mWindow.getWindowManager();
}


//1.2接着就是onCreate()，开发者主动调用setContentView(@LayoutRes int layoutResID)设置布局。
//但是我们一般是继承AppcompactActivity的，所以最后会调到setContentView(View view)的。
//然后setContentView(View view)内部会调到setContentView(View view, ViewGroup.LayoutParams params)
//最后看一下phone window的setContentView(View view, ViewGroup.LayoutParams params)的实现
public void setContentView(int layoutResID) {

    //创建DecorView
    if (mContentParent == null) {
        installDecor();
    }else{
        ...
    }
    
    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        ...
    } else {
        mContentParent.addView(view, params);
    }
    
}

//1.3接着看installDecor
private void installDecor() {

    if (mDecor == null) {
    
        //创建一个DecorView
        mDecor = generateDecor(-1);
        ...
    }else{
        //将phonewindow设置到decorview
        mDecor.setWindow(this);
    }
    
    if (mContentParent == null) {
        //加载layout
        mContentParent = generateLayout(mDecor);
        ...
    }
    ...
}

//1.3.1先看generateDecor是如何生成decor view的
protected DecorView generateDecor(int featureId) {

    ...
    //直接new一个decor view返回去
    return new DecorView(context, featureId, this, getAttributes());
}

//1.3.2再看generateLayout如何加载view的
protected ViewGroup generateLayout(DecorView decor) {

    //如果没有设置style/feature，这里默认就是R.layout.screen_simple
    layoutResource = R.layout.xxxx 
    
    //这里会执行inflater.inflate操作，然后执行addView()添加到DecorView
    mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

    ...
    //找到id为 ID_ANDROID_CONTENT(com.android.internal.R.id.content) 的ViewGroup。
    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
    ...
    return contentParent;
}

//1.4handleLaunchActivity会执行到handleResumeActivity
handleResumeActivity(){
    if (r.window == null && !a.mFinished && willBeVisible) {
    
        r.window = r.activity.getWindow();
        View decor = r.window.getDecorView();
        ...
        //ViewManager是WindowManager的父类，WindowManager有唯一的实例是WindowManagerImpl
        ViewManager wm = a.getWindowManager();
        ...
        wm.addView(decor, l);
    }
}

//1.4.1接下来看一下WindowManagerImpl.addView()的实现
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    //调用WindowManagerGlobal的addView()方法
    mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
            mContext.getUserId());
}

//1.4.2继续看WindowManagerGlobal的addView()方法
addView(){
    ...
    ViewRootImpl root;
    ...
    
    //创建一个ViewRootImpl
    root = new ViewRootImpl(view.getContext(), display);
    ...
    
    //调用ViewRootImpl的setView方法，看到这里终于走到了ViewRootImpl
    root.setView(view, wparams, panelParentView, userId);
}

//1.4.3看一下ViewRootImpl的setView()
setView(){
    ...
    //调用自身的requestLayout()，到这里，前置的内容已经走完了
    requestLayout();
}
```


### 2.绘制的大致过程

```Java
requestLayout(){
    if (!mHandlingLayoutInLayoutRequest) {
        //校验线程
        checkThread();
        //设置标识为true，设置此标识会执行performMeasure()和performLayout()
        mLayoutRequested = true;
        //开始绘制流程
        scheduleTraversals();
    }
}

void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        
        //拿到MessageQueue，然后发送同步屏障，拿到同步屏障的token
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        
        //调用Choreographer在下一帧开始绘制操作，这里执行的是mTraversalRunnable
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}

final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        //runnable就是调用doTraversal()
        doTraversal();
    }
}

void doTraversal() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
        //通过token移除同步屏障
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

        //开始绘制流程
        performTraversals();
    }
}

performTraversals(){
    ...
    
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    
    //测量
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    
    //布局
    performLayout(lp, mWidth, mHeight);
    
    //绘制
    performDraw();

    ...
}


performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
    ...
    //调用view的measure()方法
    mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    ...
}

performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {

    //调用view的layout()
    host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
}
            
performDraw() {

    //调用view的draw()
    draw()
}
```

### 3.绘制时的详细内容

- View#measure
> MeasureSpec代表的是一个32位的int类型的值，高两位的值为测量模式，而低位的值为宽高的大小。

```Java
MeasureSpec#makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,
                                  @MeasureSpecMode int mode) {
    // SDK_INT <= 17走上面
    if (sUseBrokenMakeMeasureSpec) {
        return size + mode;
    } else {
        return (size & ~MODE_MASK) | (mode & MODE_MASK);
    }
}


//通过getMode()方法去获取测量模式
MeasureSpec#getMode(int measureSpec) {
    //noinspection ResourceType
    return (measureSpec & MODE_MASK);
}

MeasureSpec中有三种测量模式: MeasureSpec.EXACTLY | MeasureSpec.UNSPECIFIED | MeasureSpec.AT_MOST

UNSPECIFIED(00)：不指定测量模式，如listview scrollview recyclerview

EXACTLY(01)：精确测量模式，如match_parent或具体的宽高，父控件可以获取到子 view精确的宽高

AT_MOST(10)：最大值模式，如wrap_content，父控件无法获取子view准确的宽高，需要子view自己去测量


//通过getSize()方法去获取测量大小
public static int getSize(int measureSpec) {
    return (measureSpec & ~MODE_MASK);
}
```
> mMeasuredWidth/mMeasuredHeight赋值

```Java
View#measure(){
    //...
    final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
    
    if (forceLayout || needsLayout) {
        mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;
        
        int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
        if (cacheIndex < 0 || sIgnoreMeasureCache) {
            onMeasure(widthMeasureSpec, heightMeasureSpec);
        }
    }
}

View#onMeasure(){
    setMeasuredDimension();
}

private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
    mMeasuredWidth = measuredWidth;
    mMeasuredHeight = measuredHeight;
    mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```

- View#layout()
```Java
//判断宽高是否有改变
View#layout(){
    boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        //调用onLayout()，View#onLayout()是空实现
        onLayout(changed, l, t, r, b);
        
        //清除标记
        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;
    }
}

View#setFrame(){
    //对比新老lrbt
    if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
        //...
        boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);
        
        //...
        invalidate(sizeChanged);
        
        //如果size有变化还会回调 onSizeChanged()
        if (sizeChanged) {
            sizeChange(newWidth, newHeight, oldWidth, oldHeight);
        }
    }
}
                

```

- View#draw()
```
View#draw(Canvas canvas){

    2 & 5默认省略

    //1. 绘制背景 Draw the background
    drawBackground(canvas);
    
    //2. 如果需要，则保存canvas的图层以备褪色 If necessary, save the canvas' layers to prepare for fading
    省略
    
    //3. 绘制view的内容 Draw view's content
    onDraw(canvas);
    
    //4.绘制view的子视图 Draw children
    dispatchDraw(canvas);
    
    //5. 如果需要，则绘制褪色区域并恢复图层If necessary, draw the fading edges and restore layers
    省略
    
    //6. 绘制装饰（如滚动条）Draw decorations (foreground,scrollbars for instance)
    onDrawForeground(canvas);
    
    //7.绘制默认焦点的高光draw the default focus highlight
    drawDefaultFocusHighlight(canvas);
}
```


## 子线程更新view
- 1.跳过`checkTread()`检查，在子线程更新UI的时候先调用`requestLayout()`方法
- 2.`checkThread()`内部校验的是`mThread != Thread.currentThread()`，所以在子线程更新UI前，调用`Looper.prepare()`，更新完后调用`Looper.loop()`。子线程弹toast也是这样处理
- 3.当`view`的宽高是固定的，且硬件加速是开启的，可以直接在子线程更新UI。原理就是：宽高都不变的话，直接更新内容是只会调用`invalidate()`，可以绕过`checkThread()`

## ViewRootImpl的测绘流程有两次测量
- 第一次测量window大小
- 测量控件树的大小

## 如何在绘制阶段拿到view的宽高
- 通过`View.post()`发送异步消息，等待
- 监听`ViewTreeObserver.OnGlobalLayoutListener`的`onGlobalLayout()`


```Java

ViewRootImpl#performTraversals(){
    //...
    host.dispatchAttachedToWindow(mAttachInfo, 0);
}

View#dispatchAttachedToWindow(){
    //...
    mAttachInfo = info;
    
    //...
    if (mRunQueue != null) {
        mRunQueue.executeActions(info.mHandler);
        mRunQueue = null;
    }
}

View#post(){
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        return attachInfo.mHandler.post(action);
    }
    //...
}
```
## 如何获取粗略的渲染耗时
> 原理就是通过同步屏障的机制，第一次post在主线程发送消息，onResume()在performTraversals()执行前执行，在performTraversals()执行后会阻塞所有同步消息，所以第二次的post会在同步屏障之后执行

```
//在onResume()里执行以下代码
final long start = System.currentTimeMillis();
    new Handler.post(new Runnable() {
        @Override
        public void run() {
            getWindow().getDecorView().post(new Runnable() {
                @Override
                public void run() {
                    Log.d(TAG, "onPause cost:" + (System.currentTimeMillis() - start));
                }
            });
        }
    });
```



## requestLayout()和invalidate()

### requestLayout()

 调用`requestLayout()`时，会设置`PFLAG_FORCE_LAYOUT`和`PFLAG_INVALIDATED`标识，然后会调用`mParent.requestLayout()`重新走绘制流程，这里的`mParent`是`ViewRootImpl`。

```Java
View#requestLayout(){
    mPrivateFlags |= PFLAG_FORCE_LAYOUT;
    mPrivateFlags |= PFLAG_INVALIDATED;
}

ViewRootImp#requestLayout(){
    mLayoutRequested = true;
    scheduleTraversals();
}
```
在`ViewRootImpl.requestLayout()`会设置`mLayoutRequested`为`true`，后面的`performTraversals()`里会根据`mLayoutRequested`的值判断是否执行`performMeasure()`和`performLayout()`。

`ViewRootImpl`调用`scheduleTraversals()`重新走绘制流程后会执行`performMeasure()`，会调用`View.measure()`方法。在`measure()`方法中有个判断:

```Java
final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
```
上面的所用的`PFLAG_FORCE_LAYOUT`正是调用`View.requestLayout()`时所设置的标识。紧接着会执行`onMeasure()`操作:

```Java
if (forceLayout || needsLayout) {
    // 清除PFLAG_MEASURED_DIMENSION_SET标识
    mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

    //因为是forceLayout，这里的cacheIndex = -1
    int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
    
    if (cacheIndex < 0 || sIgnoreMeasureCache) {
        //cacheIndex = -1，这里会走onMeasure
        onMeasure(widthMeasureSpec, heightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    } else {
        //...
    }


    //设置PFLAG_LAYOUT_REQUIRED标识
    mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
}
```
接着继续走`ViewRootImpl`，会走到`performLayout()`方法，会调用到`View.layout()`方法，在`layout`中有个判断:

```Java

boolean changed = isLayoutModeOptical(mParent) ?
        setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {

    // 调用viewgroup的onLayout()
    onLayout(changed, l, t, r, b);

    //...
    
    //清空PFLAG_LAYOUT_REQUIRED标识
    mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

    //...
}

//清空PFLAG_FORCE_LAYOUT标识
mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
```
上面的`PFLAG_LAYOUT_REQUIRED`就是调用`measure`时进入了`forceLayout`判断而设置的标识。在获取`changed`的值的时候会校验`l,r,b,t`的值:

```Java

int oldWidth = mRight - mLeft;
int oldHeight = mBottom - mTop;
int newWidth = right - left;
int newHeight = bottom - top;
//判断一下size是否有changed
boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);

//触发invalidate
invalidate(sizeChanged);


//这里还会触发View.onSizeChanged()
if (sizeChanged) {
    sizeChange(newWidth, newHeight, oldWidth, oldHeight);
}
```
上述会判断是否符合条件进行`invalidate()`

是否需要`draw()`会判断`mDirty`是否为空，为空的话就不会走`View#draw()`


### invalidate()

会往下调用`invalidateInternal()`
```Java
void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache, boolean fullInvalidate) {
    //检查线程
    checkThread();
    
    //设置为PFLAG_DIRTY
    mPrivateFlags |= PFLAG_DIRTY;

    
    if (invalidateCache) {
        //设置PFLAG_INVALIDATED
        mPrivateFlags |= PFLAG_INVALIDATED;
        mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
    }
    
    if (p != null && ai != null && l < r && t < b) {
        final Rect damage = ai.mTmpInvalRect;
        damage.set(l, t, r, b);
        p.invalidateChild(this, damage);
    }
    //...

}
```
调用`ViewGroup`的`invalidateChild`

```Java
do {
    View view = null;
    if (parent instanceof View) {
        view = (View) parent;
    }

    if (drawAnimation) {
        if (view != null) {
            view.mPrivateFlags |= PFLAG_DRAW_ANIMATION;
        } else if (parent instanceof ViewRootImpl) {
            ((ViewRootImpl) parent).mIsAnimating = true;
        }
    }

    //...

    //循环往上，最终调用ViewRootImpl的invalidateChildInParent
    parent = parent.invalidateChildInParent(location, dirty);
    
    //...
} while (parent != null);                                         
```

看`ViewRootImpl`的`invalidateChildInParent`

```Java

//检查线程
checkThread();

//判断Rect dirty是否为空
if (dirty == null) {
    invalidate();
    return null;
} else if (dirty.isEmpty() && !mIsAnimating) {
    return null;
}

//...
invalidateRectOnScreen(dirty);
```
然后会调用`scheduleTraversals()`触发重新绘制流程，但是不会触发`performMeasure()`和`performLayout()`，会触发`performDraw()`

### 总结
- `requestLayout()`会触发`measure()`和`layout()`，有可能会触发`draw()`取决于`view`的`lrbt`是否有变化。
- `invalidate()`只会触发`draw()`进行刷新`view`

