# Android 面试中级篇

本文是[Android面试题整理](https://juejin.im/post/5af82ee1f265da0b934865ba)中的一篇，结合右下角目录食用更佳，包括：

- 存储
- View
- WebView
- 性能相关
- 系统实现及原理
- 项目构建
- 功能的实现
- 概念
- 其他

## 存储

------

### 0. [Android中数据存储的方式有哪些](https://juejin.im/post/5a93ade7f265da4e6f1805d6)

> 1. File
> 2. SharedPreferences
> 3. SQlite
> 4. 网络
> 5. ContentProvider

### 1. [SharedPreference是进程同步的嘛，有没有什么方法进程同步](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F875d13458538)

> 1. 默认不是
> 2. 可以设置模式MODE_MULTI_PROCESS做到进程同步，但因为该模式有很多坑，已经被Google弃用
> 3. 官方建议使用ContentProvider

### 2. SharedPreferences commit和apply的区别

> 1. commit是同步的提交，这种方式很常用，在比较早的SDK版本中就有了。这种提交方式会阻塞调用它的线程，并且这个方法会返回boolean值告知保存是否成功（如果不成功，可以做一些补救措施）。
> 2. apply是异步的提交方式，目前Android Studio也会提示大家使用这种方式

### 3. [文件存储路径与权限](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu010937230%2Farticle%2Fdetails%2F73303034)和[权限](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fwhoislcj%2Fp%2F6137398.html)

> 1. 文件存储分为内部存储和外部存储
> 2. 内部存储
>    1. Environment.getDataDirectory() = /data //这个方法是获取内部存储的根路径
>    2. getFilesDir().getAbsolutePath() = /data/user/0/packname/files //这个方法是获取某个应用在内部存储中的files路径
>    3. getCacheDir().getAbsolutePath() = /data/user/0/packname/cache //这个方法是获取某个应用在内部存储中的cache路径
>    4. getDir(“myFile”, MODE_PRIVATE).getAbsolutePath() = /data/user/0/packname/app_myFile
> 3. 外部存储
>    1. Environment.getExternalStorageDirectory().getAbsolutePath() = /storage/emulated/0 //这个方法是获取外部存储的根路径
>    2. Environment.getExternalStoragePublicDirectory(“”).getAbsolutePath() = /storage/emulated/0 这个方法是获取外部存储的根路径 3. getExternalFilesDir(“”).getAbsolutePath() = /storage/emulated/0/Android/data/packname/files 这个方法是获取某个应用在外部存储中的files路径 4. getExternalCacheDir().getAbsolutePath() = /storage/emulated/0/Android/data/packname/cache 这个方法是获取某个应用在外部存储中的cache路径
> 4. 清楚数据和卸载APP时， 内外存储的file和cache都会被删除
> 5. 内部存储file和cache不需要权限；外部存储低版本上（19以下）file和cache需要权限，高版本不需要权限；Environment.getExternalStorageDirectory()需要权限

### 4. [SQLite](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb6634c0c8c0b)

> 1. SQLite每个数据库都是以单个文件（.db）的形式存在，这些数据都是以B-Tree的数据结构形式存储在磁盘上。
> 2. 使用SQLiteDatabase的insert，delete等方法或者execSQL方法默认都开启了事务，如果操作顺利完成才会更新.db数据库。事务的实现是依赖于名为rollback journal文件，借助这个临时文件来完成原子操作和回滚功能。在/data/data//databases/目录下看到一个和数据库同名的.db-journal文件。

> [如何对SQLite数据库中进行大量的数据插入?](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F2398aad3bd61)
> [显示的开启事务](https://link.juejin.im/?target=https%3A%2F%2Fwww.imooc.com%2Farticle%2F14069)

```
db.beginTransaction();
try {
   ...
   db.setTransactionSuccessful();
} finally {
   db.endTransaction();
  }
复制代码
```

### 5. [数据库的升级](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fqq_35114086%2Farticle%2Fdetails%2F53319093)

> 1. Android中提供了SqLiteOpenHelper类，当版本更新时，会自动调用onUpgrade方法，我们在此方法中升级
> 2. 如果修改已有表，可以才有临时表的方法：
>    1. 将已有表重命名成一个临时表
>    2. 创建新表
>    3. 拷贝
>    4. 删除临时表

### 6. 如何导入外部数据库

> 1. 把数据库db文件放在res/raw下打包进apk
> 2. 通过FileInputStream读取db文件，通过FileOutputStream将文件写入/data/data/包名/database下

## View

------

### 0. android:gravity与android:layout_gravity的区别

> 1. gravity是控制当前View内布局的位置
> 2. layout_gravity是控制View在父布局中的位置

### 1. View的绘制流程

> 从ViewRootImpl的performTraversals开始，经过measure，layout,draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。

### 2. View的measureSpec 由谁决定?顶级view呢

> 1. View的MeasureSpec由父容器的MeasureSpec和其自身的LayoutParams共同确定，
> 2. 而对于DecorView是由它的MeasureSpec由窗口尺寸和其自身的LayoutParams共同确定。

### 3. [View和ViewGroup的基本绘制流程](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu011155781%2Farticle%2Fdetails%2F52584044)

#### View

> 1. measure -> onMeasure
> 2. layout（onLayout方法是空的，因为他没有child了）
> 3. draw -> ondraw

#### ViewGroup

> 1. measure -> onMeasure (onMeasure中需要调用childView的measure计算大小)
> 2. layout -> onLayout （onLayout方法中调用childView的layout方法）
> 3. draw -> onDraw （ViewGroup一般不绘制自己，ViewGroup默认实现dispatchDraw去绘制孩子）

### 4. draw方法 大概有几个步骤

> 1. drawbackground
> 2. 如果要视图显示渐变框，这里会做一些准备工作
> 3. draw自身内容
> 4. drawChild
> 5. 如果需要, 绘制当前视图在滑动时的边框渐变效果
> 6. 绘制装饰，如滚动条

### 5. 怎么控制另外一个进程的View显示

> RemoteView：RemoteViews实现了Parcelable接口，通过binder机制传递给远程进程，进程间view的显示

### 6. [两指缩放](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F0c863bbde8eb)

> 1. 为了解决多点触控问题，android在MotionEvent中引入了pointer概念
> 2. 通过ACTION_DOWN、ACTION_POINTER_DOWN、ACTION_MOVE、ACTION_POINTER_UP、ACTION_UP来检测手机的动作
> 3. 每个手指的位置可以通过getX（pointIndex）来获得，这样我们就能判断出滑动的距离
> 4. 缩放有多种实现： 1. ImageView可以通过setImageMatrix（martix）来实现 2. 自定义View可以缩放Canvas的大小 3. 还可以设置LayoutParams来改变大小

### 7. [Scroller](https://link.juejin.im/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIwMzYwMTk1NA%3D%3D%26mid%3D2247484893%26idx%3D1%26sn%3D5874130932d4533064e40045055d0185%26chksm%3D96cda490a1ba2d86491a65f34513e50b80a5d0ccbedae644225bc0a3d262505d43381b603310%23rd)

> 1. Scroller 通常用来实现平滑的滚动
> 2. 实现平滑滚动：
>    1. 新建Scroller，并设置合适的插值器
>    2. 在View的computeScroll方法中调用scroller，查看当前应该滑动到的位置，并调用view的scrollTo或者scrollBy方法滑动
>    3. 调用Scroller的start方法开始滑动

### 8. [ScrollView时候滑动到底部](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fwang2963973852%2Farticle%2Fdetails%2F60135960)

> 1. 滑动时会调用onScrollChange方法，在该方法中监听状态
> 2. 判断childView.getMeasureHeight（总高度） == getScrollY（滑动的高度） + chilView.getHeight(可见高度)

### 9. [View事件的分发](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F38015afcdb58)

> 1. 思想：委托子View处理，子View不能处理则自己处理
> 2. 委托过程：activity -> window -> viewGroup -> view
> 3. 处理事件方法的优先级：onTouchListener > onTouchEvent > onClickListener

```
伪代码
public boolean dispatchTouchEvent(MotionEvent ev){
boolean consume = false;
if(onInterceptTouchEvent(ev)){
  consume = onTouchEvent(ev)
} else {
  consume = child.dispatchTouchEvent(ev);
}
return consume;
}

复制代码
```

> 1. 完整的事件通常包括Down、Move、Up，当down事件被拦截下来以后，move和up就不再走intercept方法，而是直接被传递给当前view处理

### 10. [什么时候执行ACTION_CANCEL](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F8360d7150786)

> 1. 一个点击或者活动事件包含ACTION_DOWN，ACTION_MOVE,ACTION_UP等
> 2. 当子View处理了ACTION_DOWN事件之后，后续的ACTION_MOVE,ACTION_UP都会直接交由这个子View处理
> 3. 如果此时父View拦截了ACTION_MOVE,ACTION_UP，那么子View会收到一个ACTION_CANCEL
> 4. 场景举例：点击一个控件，并滑动到控件外，此时次控件会收到一个ACTION_CALNCEL

### 11. 滑动冲突

> 外部拦截：重写onInterceptTouchEvent方法
> 内部拦截：重写dispatchTouchEvent方法，同时配合requestDisAllowInterceptTouchEvent方法

### 12. [ListView怎么优化](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb7741023bc6f)（举例）

> 1. 复用convertView，对convetView进行判空，当convertView不为空时重复使用，为空则初始化，从而减少了很多不必要的View的创建、减少findViewById的次数，
> 2. 采用ViewHolder模式缓存item条目的引用
> 3. 避免在getview方法中做耗时操作
> 4. 避免使用半透明或者在活动中取消半透明
> 5. 图片异步加载，待滑动停止后再加载
> 6. 局部刷新

### 13. [RecyclerView](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fxx326664162%2Farticle%2Fdetails%2F61199895)

RecyclerView 与 ListView 类似，都是通过缓存view提高性能，但是RecyclerView有更高的可定制性。下面是使用时的一些设置，通过这些设置来达到对view样式的自定义：其中1、2是必须设置的，3、4可选

> 1. 想要控制其item们的排列方式，请使用布局管理器LayoutManager
> 2. 如果要创建一个适配器，请使用RecyclerView.Adapter （Adapter通过范型的方式，帮助我们生成ViewHolder）
> 3. 想要控制Item间的间隔，请使用RecyclerView.ItemDecoration
> 4. 想要控制Item增删的动画，请使用RecyclerView.ItemAnimator
>    扩展：RecyclerView可以很方便的进行局部刷新（notifyItemChanged（））

### 14. [RecyclerView绘制流程](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fhfyd_%2Farticle%2Fdetails%2F53910631%3F_t_t_t%3D0.81394347618334)

> RecyclerView的Measure和Layout是委托LayoutManager进行的

### 15. RecyclerView的局部刷新

> 1. 调用notifyItemChange方法
> 2. 如果想刷新某个item的局部，可以有两种方法
>    1. [notifyItemRangeChanged方法](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fjdsjlzx%2Farticle%2Fdetails%2F52893469)
>    2. [根据position获取对应的ViewHolder，更新其View](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc5ca75d3a78c)

### 16. [RecyclerView的缓存](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F23339185)

> 1. RecyclerView采用四级缓存，ListView采用两级缓存
> 2. 四级缓存:
>    1. mAttachedScrap：用于屏幕内的itemView快速复用，不需要create和bind
>    2. mCacheViews：用于屏幕外的itemView快速复用，默认为两个，通过postion查找，不需要create和bind
>    3. mViewCacheExtentsion：需要用户定制，默认不实现
>    4. mRecyclerPool：默认上限5个；不需要create，需要bind；多个RecyclerView可以共用一个pool
> 3. 总结：缓存方面和ListView最大区别是mCacheViews可以缓存屏幕外的item，并且不需要重新bind

### 17. [RecyclerView 自定义LayoutManager](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fqibin0506%2Farticle%2Fdetails%2F52676670)

> 1. 重写onLayoutChildren方法
>    1. 调用detachAndScrapAttachedViews方法，缓存View
>    2. 计算并设置每个children的位置
>    3. 调用fill方法
> 2. 重写fill方法进行布局
> 3. 重写canScrollVertically和scrollVerticallyBy方法，支持滑动

### 18. [SurfaceView](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2FPicasso_L%2Farticle%2Fdetails%2F49817725)与View的区别

> 1. View主要适用于主动更新的情况下，而SurfaceView主要适用于被动更新，例如频繁地刷新；
> 2. View在主线程中对画面进行刷新，而SurfaceView通常会通过一个子线程来进行页面刷新
> 3. 而SurfaceView在底层实现机制中就已经实现了[双缓冲机制](https://link.juejin.im/?target=https%3A%2F%2Fbbs.csdn.net%2Ftopics%2F390834677)

### 19. view 的布局

布局全都继承自ViewGroup

> 1. FrameLayout(框架布局) ：没有对child view的摆布进行控制，这个布局中所有的控件都会默认出现在视图的左上角。
> 2. LinearLayout(线性布局)：横向或竖向排列内部View
> 3. RelativeLayout(相对布局)：以view的相对位置进行布局
> 4. TableLayout（表格布局）：将子元素的位置分配到行或列中，一个TableLayout由许多的TableRow组成
> 5. GridLayout:和TableLayout类似
> 6. ConstraintLayout（约束布局）：和RelativeLayout类似，还可以通过GuideLine辅助布局，适合图形化操作**推荐使用**
> 7. AbsoluteLayout（绝对布局）：已经被废弃

### 20. [线性布局 相对布局 效率哪个高](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F8a7d059da746)

> 相同层次下，因为相对布局会调用两次measure，所以线性高 当层次较多时，建议使用相对布局

### 21. View 的invalidate\postInvalidate\requestLayout方法

> 1. invalidate 会调用onDraw进行重绘，只能在主线程
> 2. postInvalidate 可以在其他线程
> 3. requestLayout会调用onLayout和onMeasure，不一定会调用onDraw

### 22. 更新UI方式

> 1. Activity.runOnUiThread(Runnable)
> 2. View.post(Runnable),View.postDelay(Runnable,long)
> 3. Handler

### 23. [postDelayed原理](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fqingtiantianqing%2Farticle%2Fdetails%2F72783952)

> 1. 不管是view的postDelayed方法，还是Handler的post方法，通过包装后最终都会走Handler的sendMessageAtTime方法
> 2. 随后会通过MessageQueue的enqueueMessage方法将message加入队列，加入时按时间排序，我们可以理解成Message是一个有序队列，时间是其排序依据
> 3. 当Looper从MessageQueue中调用next方法取出message时，如果还没有到时间，就会阻塞等待
> 4. 2中当有新的message加完成后，会检查当前有没有3中设置的阻塞，需不需要唤起，如果需要唤起则唤起

### 24. 当一个TextView的实例调用setText()方法后执行了什么

> 1. setText后会对text做一些处理，如设置AutoLink，Ellipsize等
> 2. 在合适的位置调用TextChangeListener
> 3. 调用requestLayout和invalidate方法

### 25. 自定义View执行invalidate()方法,为什么有时候不会回调onDraw()

> 1. View 的draw方法会根据很多标识位来决定是否需要调用onDraw，包括是否绑定在当前窗口等

### 26. [View 的生命周期](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F08e6dab7886e)

> 1. 构造方法
> 2. onFinishInflate：该方法当View及其子View从XML文件中加载完成后会被调用。
> 3. onVisibilityChanged
> 4. onAttachedToWindow
> 5. onMeasure
> 6. onSizeChanged
> 7. onLayout
> 8. onDraw
> 9. onWindowFocusChanged
> 10. onWindowVisibilityChanged
> 11. onDetachedFromWindow

### 27. [ListView针对多种item的缓存是如何实现的](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fwangzehuaw%2Fp%2F5383600.html)

> 1. 维护一个缓存view的数组，数组长度是 adapter的getViewItemTypeCount
> 2. 通过getItemViewType获得缓存view 的数组，取出缓存的view

### 28. [Canvas.save()跟Canvas.restore()的调用时机](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu014788227%2Farticle%2Fdetails%2F52250208)

> save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。 restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

## WebView

------

### 0. [WebView 优化](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F95d4d73be3d1)

> 1. 替换内核：不同的rom厂商对webview的优化不同，可能出现兼容性和速度问题，可以替换成X5内核等来解决兼容性问题，同时提高加载速度
> 2. WebView初始化提前：WebView初始化是一个耗时的过程，我们可以预先将WebView初始化好，例如使用全局webview
> 3. 开启缓存，可以明显提升第二次加载的速度
> 4. 优化dns解析速度：使用和api一样的域名，这样dns不用二次解析，可以提高速度
> 5. 预加载：将需要的文件资源通过native方法提前加载好或者打包进apk，需要使用时直接使用
> 6. 延迟加载：延迟加载部分资源，在界面要显示的数据加载完成后再加载，如图片资源，js等
> 7. 对于webview内存泄漏：单独开一个进程，Activity销毁时手动回收资源

### 1. WebView与Native交互

> 1. WebView中拦截网址：设置setWebViewClient，重写shouldOverrideUrlLoading
>
> 2. js与native交互
>
>    ：
>
>    1. mWebView.getSettings().setJavaScriptEnabled(true);
>    2. mWebView.addJavascriptInterface(new JSInterface(), "jsInterface");
>    3. 其他js调用Native方案：通过prompt, alert等，在webClient中重写拦截相应的方法

## 性能相关

------

### 0. [Android中ViewHolder，Handler等为什么要被声明成静态内部类（static）](https://juejin.im/post/5a951f615188257a61326473)

> 非静态内部类会持有外部类的引用，在一些情况下很可能造成内存泄漏，所以一般声明为静态内部类，但并不是说一定要生命成静态内部类。

### 1. [Android中捕获 App崩溃和重启](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fjiaweihaoku%2Farticle%2Fdetails%2F78053403)

> 1. 实现Thread.UncaughtExceptionHandler()接口，在uncaughtException方法中完成对崩溃的上报和对App的重启。
> 2. 实现自定义Application，并在Application中注册1中Handler实例。

### 2. [LruCache实现原理](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Ftianzhijiexian%2Fp%2F4248677.html)

> 最近最少使用算法：内部通过LinkedHashMap来实现

### 3. Parcelable和Serializable

> 1. 都是序列化的方式
> 2. Serializable只需实现Serializable接口即可
> 3. Parcelable需要实现Parcelable接口，并重写writeToParcel和describeContents方法，并且实现一个Creator
> 4. Serializable虽然操作简单，但是需要大量IO操作，效率慢；Parcelable自已实现封送和解封（marshalled &unmarshalled）操作不需要用反射，数据也存放在Native内存中，效率要快很多，在Android中更推荐使用Parcelable
> 5. 由于不同版本Parcelable可能存在不同，在网络和磁盘存储时，推荐使用Parcelable

### 4. 加载合适比例的Bitmap到内存中

```
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // 首先只解析图片资源的边界
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    //计算缩放值
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // 用计算出来的缩放值解析图片
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}


public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}

复制代码
```

### 5. [图片的三级缓存](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F2cd59a79ed4a)

> 1. 首次加载 Android App 时，肯定要通过网络交互来获取图片，之后我们可以将图片保存至本地SD卡和内存中
> 2. 之后运行 App 时，优先访问内存中的图片缓存
> 3. 若内存中没有，则加载本地SD卡中的图片

### 6. [App保活](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000006251859)

随着Google对Android系统的更新，以及国内厂商堆Room的定制，一些保活的手段已经失效或者不再适用，这里列举一些保活手段,实际中常常是多个方式并用

> 1. 联系Room厂商加入白名单（或者引导用户手动加入白名单）
> 2. 利用系统漏洞root进行提权，或者直接把本应用变成系统应用
> 3. Service设置成START_STICKY:kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样
> 4. 提升service优先级:通过android:priority = "1000"这个属性设置最高优先级（可能已经失效）
> 5. onDestroy方法里重启service（效果不好）
> 6. 监听广播（除了监听系统广播外，还可以利用友盟等第三方sdk做到应用相互唤起）
> 7. 启动两个进程service相互监听，互相唤起（大部分room上失效）
> 8. 在屏幕上保留1像素
> 9. 注册系统同步服务
> 10. 创建Native进程，双进程互相监听保活（5.0 以上失效）
> 11. 利用JobSheduler保活

### 7. 如何判断APP被强杀

> [这里所说的强杀场景如这篇文章](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fbce1164b83d8)
>
> 1. 在Application中定义一个static常量，赋值为－1
> 2. 在欢迎界面改为0
> 3. 在BaseActivity判断该常量的值

### 8. 常见的内存泄漏（举例）

> 1. 不恰当的使用static变量（或者向static集合中添加数据却忘了必要时移除数据）
> 2. 忘记关闭各种连接，如IO流等
> 3. [不恰当的内部类](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffrancistao%2FLearningNotes%2Fblob%2Fmaster%2FPart1%2FAndroid%2FHandler%25E5%2586%2585%25E5%25AD%2598%25E6%25B3%2584%25E6%25BC%258F%25E5%2588%2586%25E6%259E%2590%25E5%258F%258A%25E8%25A7%25A3%25E5%2586%25B3.md)：因为内部类持有外部类的引用，当内部类存活时间较长时，导致外部类也不能正确的回收（常发生在使用Handler的时候）
> 4. 不恰当的单例模式：例如错误的将某个Activity给单例持有，或者在不该使用单例的地方使用了单例
> 5. 使用错误的Context：Application 和 Activity的context生命周期不一样
> 6. webview造成的内存泄漏

### 9. OOM异常是否可以被try...catch捕获

> 1. 在发生地点可以捕获
> 2. 但是OOM往往是由于内存泄漏造成的，泄漏的部分多数情况下不在try语句块里，所以catch后不久就会再次发生OOM
> 3. 对待OOM的方案应该是找到内存泄漏的地方以及优化内存的占用

### 10. [什么是ANR，如何避免它](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fa332324956%2Farticle%2Fdetails%2F77800315)

> 1. ANR是Application Not Responding 的缩写,当应用程序无响应时，会弹出ANR窗口，让用户选择继续等待还是关闭应用。
> 2. 处理超过时间会造成ANR的地方：触摸操作等（5s）；BroadCast（10s）；Service（20s）
> 3. 避免：不要在主线程中做耗时操作

### 11. 什么情况会导致Force Close ？如何避免？能否捕获导致其的异常？

> 1. 出现运行时异常（如nullpointer/数组越界等），而我们又没有try catch捕获，可能造成Force Close
> 2. 避免：需要我们在编程时谨慎处理逻辑，提高代码健壮性。如对网络传过来的未知数据先判空，再处理；此外还可以通过静态代码检查来帮助我们提高代码质量
> 3. 此外，我们还可以在Application初始化时注册[UncaultExceptionHandler](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F84eba8efa45e)，来捕捉这些异常重启我们的程序

### 12. DDMS和TraceView

> 1. DDMS是一个程序执行查看器，在里面可以看见线程和堆栈等信息
> 2. TraceView是一个性能分析器
>    扩展： Android Studio 3 中用Profiler代替了DDMS，可以监视分析CPU，网络，内存的实时情况，更加方便

### 13. LRUCache原理

> 1. 通过LinkedHashMap实现的
> 2. LinkedHashMap的特性：LinkedHashMap是一个双向链表，当构造函数中accessOrder为true时：调用put()方法，会在集合头部添加元素时，会调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队头

### 14. 模块化的好处

> 1. 不同模块间解偶，方便复用
> 2. 单个模块支持独立编译，并行开发，提高开发和测试效率

### 15. 组件化

> 1. 组件化是已复用为目的的
> 2. 多个团队可能公用一个组件

### 16. SparseArray

> 1. SparseArray通过两个数组实现，key为int型，value为Object型
> 2. 适用于数据量较小时
> 3. 通过二分法插入和删除数据

### 17. [APP启动耗时](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc967653a9468)

> 1. 启动分为冷启动（包含初始化Application）和热启动，冷启动又可以分为第一次启动（会有额外的初始化数据库等操作）和不是第一次启动
> 2. 开发时本地可以用adb命令获取启动时常：adb shell am start -w packagename/activity
> 3. 线上可以在各个关键地方埋点统计时常

### 18. [Android启动优化](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Ff5514b1a826c)

> 1. 启动优化的目的是提高用户感知的启动速度
> 2. 可以采用TraceView等分析耗时，找出优化方向
> 3. 优化的思路：
>    1. 利用提前展示出来的Window，设置Theme，快速展示出来一个界面，给用户快速反馈的体验（启动后再改变回来）
>    2. 异步初始化一些第三方SDK
>    3. 延迟初始化
>    4. 针对性的优化算法，减少耗时

### 19. [性能调优](https://link.juejin.im/?target=http%3A%2F%2Fwww.trinea.cn%2Fandroid%2Fperformance%2F)

> 1. 性能调优包括很多方面：包括代码执行效率、网络、布局、内存、数据库、业务逻辑，打包速度等
> 2. 需要针对具体问题具体分析，找到性能瓶颈：
> 3. 善于借助工具，例如使用traceview跟踪，或者打点统计，profiler，leak canary等

#### [网络优化](https://link.juejin.im/?target=http%3A%2F%2Fwww.trinea.cn%2Fandroid%2Fmobile-performance-optimization%2F)：

> 1. 不用域名,用ip直连
> 2. 请求合并与拆分
> 3. 请求数据的缩小：删除无用数据，开启Gzip压缩
> 4. 精简的数据格式：json/webp/根据设备和网络环境的不同采用不同分辨率图片
> 5. 数据缓存
> 6. 预加载
> 7. 连接的复用：使用http2.0 (效率提升30%)
>    扩展：[JD Android客户端网络性能调优之HTTP/2协议升级](https://link.juejin.im/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FN16jX2zRT3mlaJF4ixDhMg)

#### [布局优化](https://link.juejin.im/?target=http%3A%2F%2Fwww.trinea.cn%2Fandroid%2Flayout-performance%2F)

> 1. 合理的使用include/merge/viewstub等
> 2. 减少布局层次，减少不必要的view和节点
> 3. 使用布局缓存，避免重复inflate
>    其他：可以用hierarchy viewer等工具进行分析

#### [代码优化](https://link.juejin.im/?target=http%3A%2F%2Fwww.trinea.cn%2Fandroid%2Fjava-android-performance%2F)

> 1. 降低执行时间：如优化算法和数据结构/利用多线程/缓存(包括对象缓存、线程池、IO 缓存、网络缓存等)/JNI/需求优化
> 2. 同步改异步：例如使用surfaceview
> 3. 错峰：提前或者延迟操作，比如启动中第三方sdk的加载

#### [APK瘦身](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fvfush%2Farticle%2Fdetails%2F52266843)

> 1. 分析APK
>    1. 使用Android Studio的分析器分析apk结构
>    2. 使用lint分析无用代码和资源
>    3. 或者使用第三方工具，如NimbleDroid/ClassShark
> 2. 删除无用资源
>    1. 对无用资源和代码删除
>    2. 优化结构，对重复资源去重
>    3. 对依赖去重，依赖多个功能类似的sdk时，只保留一个
>    4. 去除不需要的依赖，如语言support包可能包含多种语言，配置只保留我们需要的资源等
>    5. 开启gradle的ProGuard/Code shrinking/minifyEnabled等，自动去除不需要的资源和代码】
> 3. 压缩已用资源
>    1. 选取适当的图片格式，如webp
>    2. 对图片进行压缩
>    3. 选取合适的依赖包，而不是直接依赖V7包等
>    4. 使用微信的打包插件AndResGuard对图片进行压缩
>    5. 使用facebook的ReDex对dex文件进行压缩
> 4. 通过网络按需加载

### 20. 有什么提高编译速度的方法

> 1. 提高电脑配置
> 2. 优化Android studio配置
>    1. 加大内存；
>    2. 使用守护进程；
>    3. 开启instant Run
>    4. 使用离线gradle
>    5. 使用新版的gradle
> 3. 优化项目
>    1. 使用第三方编译如[阿里的Freeline](https://www.freelinebuild.com/docs/zh_cn/###
>    2. debug模式下可以不开启混淆等
>    3. [模块化](https://link.juejin.im/?target=http%3A%2F%2Fstormzhang.com%2Fandroid%2F2015%2F03%2F01%2Fandroid-reference-local-aar%2F)，多模块时使用aar包避免反复编译

### 21. [延迟加载的实现](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fa0e242d57360)

> 1. myHandler.postDelayed(mLoadingRunnable, DEALY_TIME);
> 2. 

```
 getWindow().getDecorView().post(new Runnable() {
    @Override
   public void run() {
        myHandler.post(mLoadingRunnable);
   }
  });
复制代码
```

> 1. [监听MessageQueue，当空闲时执行任务](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F545cf65c4f5e)

```
 // 拿到主线程的MessageQueue
 Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() { 
      @Override public boolean queueIdle() { 
           // 在这里去处理你想延时加载的东西 
           delayLoad(); 
           
           // 最后返回false，后续不用再监听了。
           return false; 
           } 
  });

复制代码
```

### 22. [ArrayMap](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fba65a0aeec44)

> 1. ArrayMap 是Android中的一种用时间换空间的容器，用来替代HashMap
> 2. 不适合大量数据或者有大量删除
> 3. 采用二分查找
> 4. 原理： 0. 使用两个数组进行存储
>    1. 数组1储存key的HashCode
>    2. 数组2储存key和value；value位置为key位置左移1位再加1
>    3. hash冲突的解决：寻找下一个空位，因为查询时会比较hashcode和key，所以不会有问题

## 系统实现及原理

------

### 0. AsyncTask

> 1. 为我们封装了thread和handler，并在内部实现了线程池，是一个轻量级的线程方案
> 2. 缺点：不同版本实现方式可能不一样，例如线程池有可能是串行的（最新版本是并行的）
> 3. 缺点：可定制化程度不高，例如我们不能很方便地cancel线程

### 1. [AsyncTask是顺序执行么](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F79cc3c5fc9a3)，for循环中执行200次new AsyncTask并execute，会有异常吗

> 1. 不同版本的AsyncTask 不一样，最新版本中是串行的
> 2. 不会，因为串行执行时用的是一个ArrayDeque来存放Runnable
> 3. 扩展：AsyncTask内部有并行的ThreadPoolExecutor，储存队列用的是LinkedBlockingQueue，大小是128；如果我们并行执行，会抛出运行时异常

### 2. [Android 渲染机制](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F1ef2a9e5aa91)

> 1. CPU对视图进行必要的计算：measure等
> 2. 通过OpenGL 将CPU 处理过的数据交给GPU
> 3. GPU进行栅格化并存入缓存
> 4. Android 系统每隔16.6ms发出一个垂直同步信号，通知渲染

### 3. [Android中Application和Activity的Context对象的区别](https://juejin.im/post/5a9514e25188257a865d9b5a)

> 1. 生命周期不一样
> 2. Application 不能showDialog
> 3. Application startActivity时必须new一个Task
> 4. Application layoutInflate直接使用默认主题，可能与当前主题不一样

### 4. [Zygote的启动过程](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fedeac788d60c)

> 1. 注册Zygote的socket监听端口，应用接收启动应用程序的消息
> 2. 调用preload()方法加载系统资源，包括预加载类，Framework资源等
> 3. 调用startSystemServer()方法启动SystemServer进程
> 4. 调用runSelectLoop()方法进入监听和接收消息循环

### 5. [Handler、Looper、Message、MessageQueue](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu012827296%2Farticle%2Fdetails%2F51236614)

它们的存在主要是为了线程间通信，通信的方式为：

> 1. 在线程中调用Looper.prepare方法，生成一个looper与当前线程绑定（在生成looper的过程中，其构造方法在其内部创建了一个MessageQueue）
> 2. 调用looper.loop方法，使当前线程循环读取MessageQueue中的message，并调用 msg.target.dispatchMessage方法，交由target处理（这里target是一个Handler实例）
> 3. new 一个Handler，在创建Handler时，需要为他指定looper（若不指定则是当前线程的looper），Handler会取出looper中的MessageQueue也作为自己的MessageQueue。
> 4. 调用Handler的sendMessage方法发送Message信息（这个方法将Message的target设置成当前handler，并把它加入到MessageQueue中）

### 6. [为什么主线程Looper.loop不会ANR](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fcfe50b8b0a41)

> 1. ANR是应用程序无响应，原因是有事件在主线程运行时间过长造成新的事件无法处理，或者当前事件运行时间太长
> 2. Looper.loop会循环处理到来的Message，当MessageQueue为空是，线程处于阻塞状态，释放cpu资源

### 7. [Messenger](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu011974987%2Farticle%2Fdetails%2F51243539) 和 AIDL

> 都是进城间通信的方式，AIDL使用Binder通信，Messenger通过AIDL实现
>
> Messenger原理：Handler内部持有一个IMessenger实例，IMssenger时一个aidl的接口，Messenger初始化时这个IMssenger实例也传给了Messenger
>
> [区别](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fjiwangkailai02%2Farticle%2Fdetails%2F48098087)：1.Messenger是对AIDL的封装，使用简单；2.Messenger通过Handler处理传过来的Message，只能运行在一个线程中，我们在AIDL中可以运行多个线程；3.Messenger客户端调取服务方法的结果只能异步回调给客户端，AIDL可以同步回调

### 8. [Binder](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fadaa1a39a274)

瞎jj写点凑个数吧

> 1. Binder是Android中的一种进程间通信方式
> 2. Binder通信是通过驱动来实现的，原理是在内核空间中进行内存映射，以为只有一次内存拷贝，所以速度较快
> 3. Binder会验证权限，鉴定UID/PID来验证身份，保证了进程通信的安全性
> 4. 系统的Service会想ServiceManager注册，使用时向ServiceManager获取
> 5. Service/Client同ServiceManager通信的过程本身也是通过binder驱动实现的
> 6. android中[Service](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F04ad8a12d808)的bind通信是通过ActivityManagerService实现的
> 7. Binder中的代理模式：
>    1. 客户端代理对象和服务端实现统一接口
>    2. 客户端获取服务端的引用，如果位于同一进程，那么获取的是服务端本身，如果是不同进程，获取到的是服务端的代理
>    3. 通过代理请向服务端请求
>    4. 服务端接收请求并处理，返回给客户端

### 9. [AIDL](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu011240877%2Farticle%2Fdetails%2F72765136)

> 1. 全称：Android Interface Define Language（Android接口定义语言）
> 2. 目的是为了进行进程间通信
> 3. 使用方式：定义aidl文件，编译器编译时会帮我们生成对应的JAVA代码；通过调用生成的java代码，来进行进程间通信
> 4. 原理：通过Binder方式进行通信

### 10. [Window、WindowManager以及Activity](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fyhaolpz%2Farticle%2Fdetails%2F68936932)

> 1. Window 0. Window有三类：系统Window、应用Window、子Window
>    1. Window是接口，具体实现类是PhoneWindow
>    2. Window 是一个抽象概念，我们并不能直接操作window
>    3. Activity在创建的时候attach方法中会创建Window并使之与Activity关联
>    4. Window中会创建Decorview，并通过ViewRootImpl与View交互
> 2. WindowManager 0. 在Activity启动时，handleResumeActivity方法中启动activity的时候，会将主窗口加入到WindowManager中
>    1. 我们并不能直接操作window，而是通过WindowManager
>    2. WindowManagerImpl是其实现类，他将view增删改的操作交给 WindowManagerGlobal处理
>    3. WindowManagerGlobal 中会调用 ViewRootImpl的方法
>    4. ViewRootImpl通过IWindowSession与WindowManagerService交互

### 11. 理解Window和WindowManager

> 1. Window用于显示View和接收各种事件，Window有三种类型：应用Window(每个Activity对应一个Window)、子Window(不能单独存在，附属于特定Window)、系统window(Toast和状态栏)
> 2. Window分层级，应用Window在1-99、子Window在1000-1999、系统Window在2000-2999.WindowManager提供了增删改View三个功能。
> 3. Window是个抽象概念：每一个Window对应着一个View和ViewRootImpl，Window通过ViewRootImpl来和View建立联系，View是Window存在的实体，只能通过WindowManager来访问Window。
> 4. WindowManager的实现是WindowManagerImpl其再委托给WindowManagerGlobal来对Window进行操作，其中有四个List分别储存对应的View、ViewRootImpl、WindowManger.LayoutParams和正在被删除的View
> 5. Window的实体是存在于远端的WindowMangerService中，所以增删改Window在本端是修改上面的几个List然后通过ViewRootImpl重绘View，通过WindowSession(每个应用一个)在远端修改Window。
> 6. Activity创建Window：Activity会在attach()中创建Window并设置其回调(onAttachedToWindow()、dispatchTouchEvent()),Activity的Window是由Policy类创建PhoneWindow实现的。然后通过Activity#setContentView()调用PhoneWindow的setContentView。

### 12. [Window添加的过程](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fkebelzc24%2Farticle%2Fdetails%2F53888931)

> 1. ActivityThread做为APP的入口，会执行 handleLaunchActivity -> performLaunchActivity
> 2. performLaunchActivity中通过ClassLoader创建Activity对象 -> 调用Activity.attach方法 -> callActivityOnCreate
> 3. 在Activity.attach方法中会创建一个Window实例PhoneWindow，并将activity作为callback传递给window
> 4. 在Activity的onCreate方法中，会调用setContentView，setContentView会调用PhoneWindow 的 setContentView
> 5. PhoneWindow 的 setContentView会创建DecorView，并把我们自己设置的ContentView和DecorView绑定
> 6. performLaunchActivity至此走完，之后的performResumeActivity会调用handleResumeActivity方法
> 7. 在handleResumeActivity中，会调用Activity的getWindowManager()获取一个WindowManager，接着调用WindowManager的addView方法
> 8. addView实际执行的是WindowManagerGlobal的addView()，这里会创建一个ViewRootImpl，并调用ViewRootImpl的setView方法
> 9. 在setView这个方法内部，会通过跨进程的方式向WMS（WindowManagerService）发起一个调用，从而将DecorView最终添加到Window上

### 13. APK的安装流程



![img](https://user-gold-cdn.xitu.io/2018/5/4/16329f99d3a4b73a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



> 1. 解压文件到data/app目录下
> 2. 资源管理器加载资源文件
> 3. 解析解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。
> 4. 然后对dex文件进行优化，并保存在dalvik-cache目录下。
> 5. 将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。
> 6. 安装完成后，发送广播。

### 14. [双亲委托模式类加载的过程](https://link.juejin.im/?target=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-lo-classloader%2F)

> 1. 调用当前类加载器的loadClass加载类
> 2. loadClass中先调用findLoadedClass查看类是否已加载，已加载->over
> 3. 还没有加载，调用父类加载器的loadClass，父类加载器加载完成 -> over
> 4. 如果父类加载器没有加载，调用本类加载器的findClass,并在findClass中调用·defineClass方法加载

### 15. Android中ClassLoader

> 1. Android中加载的是Dex文件
> 2. ClassLoader 是个抽象类，其具体实现的子类有 BaseDexClassLoader 和SecureClassLoader，（SecureClassLoader 的子类是 URLClassLoader ，其只能用来加载 jar 文件，这在 Android 的 Dalvik/ART 上没法使用的）
> 3. BaseDexClassLoader 的子类是 PathClassLoader 和 DexClassLoader 。
> 4. PathClassLoader 在应用启动时创建，从 data/app/… 安装目录下加载 apk 文件。
> 5. DexClassLoader 则没有此限制，可以从 SD 卡或网络加载包含 class.dex 的 .jar 和 .apk 文件，这也是插件化和热修复的基础

## 项目构建

------

### 0. [点击AndroidStudio的build按钮后发生了什么](https://juejin.im/post/5a939fdd5188257a585109ff)

build过程即执行gradle task 打包生成apk的过程：

> 1. 通过appt工具，将资源文件生成R.java文件；将aild文件转换成对应的java文件
> 2. 编译java文件，生成.class文件
> 3. 将.class文件转换成Android虚拟机支持的.dex文件
> 4. 通过apkbuilder将dex文件和编译后的资源文件生成apk文件
> 5. 对apk进行签名和对齐

### 1. Android Debug和Release状态的不同

调试模式允许我们为优化代码而添加许多额外的功能，这些功能在Release时都应该去掉；Release包可以为了安全等做一些额外的优化，这些优化可能比较耗时，在Debug时是不需要的

> 1. log日志只在debug时输入，release时应该关掉（为了安全）
> 2. 签名/混淆/压缩等在debug编译时可以加入，减少打包时间
> 3. 可以在debug包中加入一些额外的功能辅助我们开发，如直接打印网络请求的控件，内存泄漏检测工具LeakCanary等
> 4. 在打Release包时，除了混淆等操作，往往还需要加固操作，保证APP的安全

### 2. [Dalvik和Art](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fjason0539%2Farticle%2Fdetails%2F50440669)

> 1. Dalvik 是 Android 中使用的虚拟机，执行dex字节码
> 2. Dalvik 与 JVM 相比
>    1. JVM执行class字节码文件，Dalvik执行dex字节码文件，dex文件做了优化，提及更小
>    2. Dalvik是基于寄存器的，VM基于栈
> 3. Art是Dalvik虚拟机的升级版，Dalvik是解释型的，Art是翻译型的
>    1. Dalvik虚拟机执行的是dex字节码，ART虚拟机执行的是本地机器码
>    2. 相对于Dalvik，Art安装占用内存更大，安装时间更长，但是运行速度会有提升

### 3. 解决方法数超过65535的方法

> 1. 代码混淆(原理是减小方法数)
> 2. sdk裁减(原理是减小方法数)
> 3. multi-dex（原理是打包成多个dex）

### 4. [multi-dex](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F79a14d340cb0) 问题

> 引入multi-dex后，在5.0以下手机上，第一次安装后启动速度可能变慢甚至anr，需要进行优化:如单独开一个线程；修改keep文件等

### 5. [App签名](https://link.juejin.im/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F6GV5HFnzZnY_qJlV9scVrw)

> 1. 原理：先计算出hash值，再对hash值进行非对称加密
> 2. V1版本签名生成的APK中与签名有关的文件：
>    1. MANIFEST.MF: jar 包的文件清单，在 apk 中我们用来记录所有非目录文件的 数据指纹
>    2. CERT.SF:根据MANIFEST.MF生成的文件
>    3. CERT.RSA：这里会把之前生成的CERT.SF文件，用私钥计算出签名, 然后将签名以及包含公钥信息的数字证书一同写入CERT.RSA 中保存
> 3. V1版本存在安全漏洞，google推出了v2版

### 6. [常用的adb 和 adb shell命令](https://link.juejin.im/?target=http%3A%2F%2Fwww.jb51.net%2Farticle%2F112562.htm)

> 查看当前连接的设备：adb devices
> 结束adb连接： adb kill-server
> 安装apk： adb install test.apk
> 从手机获取文件和推送文件到手机：adb push <本地文件> <远程路径> ； adb pull <远程路径> <本地路径>
> 获取log信息：adb logcat > log.txt

> 启动Activity： adb shell am start -n 包名/包名＋类名
> 显示系统Activity栈信息：adb shell dumpsys activity
> 发送广播：adb shell am broadcast -a "android.intent.action.AdupsFota.WriteCommandReceiver"
> 查看进程信息：adb shell ps <package_name|PID>
> 杀掉某个进程：adb shell kill pidNumber
> 查看内存占用：adb shell dumpsys meminfo <package_name|PID>

### 7. jar和aar的区别

> Jar包里面只有代码，aar里面不光有代码还包括代码还包括资源文件，比如 drawable 文件，xml 资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度

### 8. [不同的CPU架构对APP的影响](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fcb05698a1968)

> 1. cpu的架构有armeabi、armeabi-v7a、x86等
> 2. 针对不同的CPU，使用不同的so包，可以使性能最大化
> 3. 如果a.so提供了armeabi、armeabi-v7a、x86格式，那么b.so也要提供这几个格式，否则可能崩溃
> 4. 当没有对应cpu的so时，会选择其他so，但执行速度会变慢：当一个应用安装在设备上，只有该设备支持的CPU架构对应的.so文件会被安装。在x86设备上，libs/x86目录中如果存在.so文件的话，会被安装，如果不存在，则会选择armeabi-v7a中的.so文件，如果也不存在，则选择armeabi目录中的.so文件（因为x86设备也支持armeabi-v7a和armeabi）

### 9. [compileSdkVersion，minSdkVersion，targetSdkVersion](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu010286855%2Farticle%2Fdetails%2F54691614) 都是啥

> 1. compileSdkVersion ：编译所依赖的版本，它可以让我们在写代码时调用最新的api，告知我们过时的api
> 2. minSdkVersion：最小的可安装此App的版本，意味着我们不用做低于此版本的兼容
> 3. targetSdkVersion: 目标版本，可以让我们虽然运行在最新的手机上，但是行为和target版本一致，比如：如果targetSdkVersion小于Android 6.0，那么即使我们的app运行在6.0系统上，也不需要运行时权限

### 10. 低版本SDK实现高版本api

> 1. 使用@TargetApi 来标明api版本，这样编译器就不会报错了
> 2. 在代码逻辑中判断版本，在低版本上调用替代api或自己实现的算法。

## 部分功能的实现

------

### 0. [怎样退出终止自己的APP](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Ffzkf9225%2Farticle%2Fdetails%2F73480469)

> 1. 记录启动的activity
> 2. 需要退出时调用存活activity的finish方法，并调用System.exit(0)方法

### 1. Android中启动线程的几种方式

> 1. java中可以用实现Runnable接口、实现Callable接口、继承Thread类三种方式
> 2. Android中还可以用[AsyncTask](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F3b839d7a3fcf)、[HandlerThread](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F4a57de01c8f5)、[IntendService](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fmatrix_xu%2Farticle%2Fdetails%2F7974393)

### 2. [长链接+心跳包](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fu010072711%2Farticle%2Fdetails%2F76099776)

> 1. 长连接：App 与服务器建立一个生命周期很长的连接，服务器通过push向App推送消息
> 2. 心跳包：App 每隔一段时间就会向服务器查询是否有新的消息
> 3. 长连接可能因为各种原因被打断，心跳包接收消息可能不及时，所以我们可以采取长连接+心跳包的方式：通过Socket建立一个长连接，并通过心跳包检测这个长连接是否存活，长连接中断的话则重新建立

### 3. [Android 中XML的解析](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fe636f4f8487b)

> Androi中主要有DOM，SAX，PULL三种方式
> DOM将文件都加载到内存中，比较消耗内存；SAX和PULL节省内存，PULL使用比SAX更简单

### 4. 系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。

> 1. 指定浏览器：intent.setClassName(“com.android.browser”,”com.android.browser.BrowserActivity”);
> 2. 指定网址： Uri uriBrowsers = Uri.parse(“http://www.sina.com.cn”); intent.setData(uriBrowsers);

### 5. java中如何引用本地语言

> 可以用JNI（java native interface java 本地接口）接口

### 6. [JNI的使用方式](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fkevindgk%2Farticle%2Fdetails%2F52813258)

> 1. 下载NDK，配置环境变量，配置gradle文件
>
> 2. JAVA中声明native 方法如private native String printJNI(String inputStr);
>
> 3. 生成或写对应的头文件
>
> 4. 编写对应文件实现代码
>
> 5. 编译成so文件
>
> 6. 使用
>
> 7. 扩展：
>
>    native调用java代码
>
>    1. 获取你需要访问的Java对象的类

```
jclass cls = (*env)->GetObjectClass(env, obj);       //使用GetObjectClass方法获取obj对应的jclass。   
class cls = (*env)->FindClass(“android/util/log”) //直接搜索类名，需要是static修饰的类。  
复制代码
```

> 1. 获取MethodID：

```
methodID mid = (*env)->GetMethodID(env, cls, "callback", "(I)V"); //GetStaticMethodID(…)，获取静态方法的ID使用GetMethdoID方法获取你要使用的方法的MethdoID  
复制代码
```

> 1. 调用：

```
    (*env)->CallVoidMethod(env, obj, mid, depth);// CallStaticIntMethod(….) ， 调用静态方法  
复制代码
```

### 7. NDK是什么

> NDK 是Native Development Kit 的缩写，是一些列工具的集合，帮助开发者迅速的开发C/C++的动态库

### 8. [什么是JVM](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000002579346)

> 1. JVM是JAVA虚拟机，保证了java语言的跨平台性，是编译后的 Java 程序（.class文件）和硬件系统之间的接口
> 2. JVM = 类加载器 classloader + 执行引擎 execution engine + 运行时数据区域 runtime data area

### 9. [视频加密](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgwuhaolin%2Fblog%2Fissues%2F10)

视频加密根据场景的不同有很多种方式

> 1. 如仅对地址加密，可以起到防盗链的目的，可以与其他方法一起使用
> 2. 对整个文件加密，加解密时间长，不实用
> 3. 对文件的头中尾加密，播放器可以直接跳过，破解简单,不实用
> 4. 对视频流加密([基于苹果HLS协议的加密](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fhauk0101%2Fvideo-hls-encrypt) [基于RTPE协议](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgwuhaolin%2Fblog%2Fissues%2F10))
> 5. 关键帧加密

### 10. [绘制印章](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Floser_li%2Farticle%2Fdetails%2F48005683)

> 1. 创建一个bitmap，拿到canvas
> 2. 在canvas上绘制圆，绘制五角星，绘制文字，返回bitmap

### 11. [文字阴影和描边的实现](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Ffigo0423%2Farticle%2Fdetails%2F51464116)

> 1. 阴影：shadow属性
> 2. 描边：两个TextView叠加；或者重写onDraw方法

### 12. [Android生成设备唯一标识符](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb6f4b0aca6b0)

> 选取 DeviceId，AndroidId，Serial Number，Mac，蓝牙地址等中的一个或者几个作为种子，生成UUID。

### 13. [实现客户端的自动登录](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fxiaopihair123%2Farticle%2Fdetails%2F53150201)

> 1. 第一次登录时保存两个token，一个长效一个短效
> 2. 短效token用于每次网络请求的用户标识
> 3. 长效token用于当短效token失效时自动登录，重新获取token

### 14. [Android如何在不压缩的情况下加载高清大图](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb0e2be9e0f8c)

> 使用BitmapRegionDecoder

### 15. SSL证书的验证

> 1. 在使用Https时，我们需要对SSL证书做验证以确保有效
>
> 2. 证书需要验证证书有效性，时效，域名等
>
> 3. Android中WebView可以重写onReceivedSslError方法来处理ssl证书不对时的情况
>
> 4. OkHttp设置证书验证
>
>    ：
>
>    1. 验证可以是双向的，也可以是单向的
>    2. 单向：将服务器对应的Server.cer文件打包进Apk中，通过cer文件生成SSLSocketFactory，并将其设置给okHttpClient
>    3. 双向：用Server.cer和Client.key生成SSLSocketFactory，并将其设置给okHttpClient

### 16. [计算一张100px*100px的图片在内存中会占用多大内存](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2FYuangPong%2Fp%2F6694512.html)

> 1. 内存大小 = 100*100*像素点大小
> 2. 像素点大小和编码方式有关：ARGB_8888占8+8+8+8=32bit；ARGB_4444占4+4+4+4 = 16bit；

### 17. [如何实现动态代理](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fhello2mao%2Farticle%2Fdetails%2F52346205)

> 1. 创建一个实现InvocationHandler接口的类，它必须实现invoke方法
> 2. 调用Proxy的静态方法newProxyInstance，创建一个代理类

### 18. Android中有哪些方法实现定时和延时任务？它们的适用场景是什么？

> 1. 倒计时类:用CountDownTimer
> 2. 延迟类: 1. CountDownTimer，可巧妙的将countDownInterval设成和millisInFuture一样，这样就只会调用一次onTick和一次onFinish 2. handler.sendMessageDelayed,可参考CountDownTimer的内部实现，简化一下，个人比较推荐这个 3. TimerTask，代码写起来比较乱 4. Thread.sleep，感觉这种不太好
> 3. 定时类: 1. 参照延迟类的，自己计算好要延迟多少时间 2. handler.sendMessageAtTime 3. AlarmManager，适用于定时比较长远的时间，例如闹铃

## 概念

------

### 0. Json有什么优势

有比较才会有优势，我们通常将Json与Xml进行比较，Json更加轻量。我觉得在某些程度上讲，这是一个仁者见仁智者见智的问题。例如有些人认为Json相比Xml更易读，有些人责认为不然，这里大致列举几条，仅供参考

> 1. 结构简单，可读性更强，读写更加容易
> 2. 格式是压缩的，占用带宽小
> 3. 支持多种语言
> 4. 因为JSON格式能够直接为服务器端代码使用,大大简化了服务器端和客户端的代码开发量

### 1. [MVC](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2FClaire6649%2Fp%2F6091061.html)

> MVC是model,view,controller的缩写

> 1. 模型（model）对象：是应用程序的主体部分，所有的业务逻辑都应该写在该层;
> 2. 视图（view）对象(**对应Android中的布局xml文件**)：是应用程序中负责生成用户界面的部分。也是在整个mvc架构中用户唯一可以看到的一层，接收用户的输入，显示处理结果; 
> 3. 控制器（control）对象（**对应Android中的Activity**）：是根据用户的输入，控制用户界面数据显示及更新model对象状态的部分，控制器更重要的一种导航功能，响应用户出发的相关事件，交给m层处理。 扩展：和MVP最大的区别是View和Model可以直接交互

### 2. [什么是控制反转（IOC Inversion of Control）](https://link.juejin.im/?target=http%3A%2F%2Fwww.jb51.net%2Farticle%2F118316.htm)

> 1. 在Java开发中，IoC意 味着将你设计好的类交给系统去控制，而不是在你的类内部控制
> 2. 是框架的重要特征
> 3. Android中Activity 的生命周期都是框架控制的，是一种控制反转

### 3. [Android中的IOC（控制反转）框架](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F3968ffabdf9d)

> 1. 控制反转：将对象的创建交给框架去做
> 2. 常用的框架：ButterNife/Android Annotation
> 3. 2中两个框架都是通过java的注释框架实现的，并且都是作用在编译期

### 4. 请解释下Android程序运行时权限与文件系统权限的区别

> 1. 运行时权限是APP启动后由Android虚拟机（如Dalvik）控制的
> 2. 文件系统权限是Linux内核授权

### 5. [AOP 面向切面编程](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fganchuanpu%2Fp%2F8594877.html)

> 1. 面向切面编程是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术
> 2. 例如很多功能需要先登陆，登陆在这里就是一个切面。

### 6. MVP 架构中 Presenter 定义为接口有什么好处

> 1. 在goole官方的demo中，通过一个Contract把将View和Presenter管理起来，强化其一一对应的关系，便于操作
> 2. [但是也有人认为不应该将Presenter定义为接口]http://www.infoq.com/cn/articles/do-not-give-the-prensenter-in-mvp-interface)，因为这并不会方便测试，还会增加复杂性

## 其他

------

### 0. [Android中的几种动画](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fldq2016%2Fp%2F5407061.html)

> 1. 普通动画（视图动画、补间动画）
> 2. [属性动画](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F2412d00a0ce4)
> 3. 帧动画
> 4. 物理动画（Android 8.0）

### 1. [HttpClient和HttpURLConnection的区别](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fa32d6980227b)

#### HttpClient

> 1. Apache公司提供的库
> 2. 拥有丰富的API，但也因为这个原因，在不破坏兼容性的前提下，其庞大的API也使人难以改进
> 3. Android 6.0中抛弃了Http Client，替换成OkHttp

#### HttpURLConnection

> 1. Sun公司提供的库
> 2. 功能比较简单，可拓展性强
> 3. 直接支持GZIP压缩，并且在Android 4.0 以上支持cache缓存，提高了网络效率

### 2. Intent

> 1. Intent是Android中的信使，可以启动Activity，Service等
> 2. Intent可以设置的几项值：Action, Category, Data/Type,Component
> 3. 当设定Component时，是显式调用，其余是隐式调用

### 3. [IntentFilter](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fmynameishuangshuai%2Farticle%2Fdetails%2F51673273)

> 1. IntentFilter 在AndroidMainifest中注册，用来帮助系统选出用户定义的隐式Intent对应的Activity /Service等
> 2. Android是通过Intent的action、data、category这三个属性来进行匹配判断的
>    1. action：隐式启动需要给Intent设置Action，如果没有设置这条匹配则自动通过；必须给IntentFilter设置一个action
>    2. data：如果Intent没有提供type，系统将从data中得到数据类型。同action类似，只要Intent的data只要与Intent Filter中的任一个data声明完全相同，data方面就完全匹配成功。
>    3. category：Intent的Category可以有多个，每一个都需要和IntentFilter匹配才能算匹配上，不设置Intent的category时是默认的（DEFAULT）
>    4. 总结：只有一个Intent的action、data、category匹配上IntenFilter中的一组数据时，才算匹配成功。

### 4. Intent/Bundle支持传送哪种类型的数据

> 1. 基本类型及其数组
> 2. 实现了Serializable或者Parcelable的类型及其数组

### 5. Manifest.xml文件中主要包括哪些信息

> 1. manifest：根节点，描述了包名，版本号等。
> 2. application：包含package中application级别组件声明的根节点。
> 3. activity：Activity是用来与用户交互的主要工具。
> 4. receiver：IntentReceiver能使的application获得数据的改变或者发生的操作，即使它当前不在运行。
> 5. service：Service是能在后台运行任意时间的组件。
> 6. provider：ContentProvider是用来管理持久化数据并发布给其他应用程序使用的组件。
> 7. uses-permission：请求你的package正常运作所需赋予的安全许可。
> 8. permission： 声明了安全许可来限制哪些程序能你package中的组件和功能。
> 9. uses-feature:使用到的硬件信息，如nfc
> 10. upports-screens：支持的屏幕类型
> 11. meta-data：data数据
> 12. [instrumentation](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fa19891024%2Farticle%2Fdetails%2F54342799)：声明了用来测试此package或其他package指令组件的代码。

### 6. [dp, dip, dpi, px, sp是什么意思](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fbinyao02123202%2Farticle%2Fdetails%2F8090607)

> 1. dp = dip（device independent pixels）,是设备独立像素
> 2. sp:scaled pixels(放大像素)，主要用于字体显示。
> 3. px（pixel）：像素
> 4. dpi（dot per inch）

### 7. [dp与px的换算公式](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fbenbenxiongyuan%2Farticle%2Fdetails%2F52920746)

> 1. px = dp*像素密度/160

```
 public static int dp2px(Context context, float dpValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dpValue * scale + 0.5f);
    }

复制代码
```

### 8. [layout-sw400dp, layout-w400dp分别代表什么意思](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fwxx614817%2Farticle%2Fdetails%2F50975265)

> 1. layout-w400dp:当屏幕相对宽度大于400dp时，来这里取layout（与横竖屏有关）
> 2. layout-sw400dp:当屏幕绝对宽度大于400dp时，来这里取layout（与横竖屏无关）

### 9. [Android 样式和主题](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fahou2468%2Farticle%2Fdetails%2F78965839)

> 1. 样式（Styles）:可以理解成是针对View或者窗口(Window)设置外观或者格式的一个属性集合
> 2. 主题（Themes）：主题相比单个视图而言，是应用到整个 Activity 或者 application 的样式
> 3. 区别：
>    1. Theme作用域是Activity或者Application，Stytle针对View或者窗口(Window)
>    2. 某些主题样式不可以在View中使用，例如"@android:style/Theme.NoTitleBar" 等 扩展： 属性（Attributes）:你也可以将单个属性应用到 Android 样式上,通常会在自定义View 的时候，自定义属性。

### 10. Android P的新特性（2018/5/5）

> 1. Goole 下个Android版本的预览已经放出，代号p
> 2. 支持wifi室内定位
> 3. 适配刘海屏
> 4. 通知栏改进：可以显示对话，附加照片和表情等
> 5. 多摄像头API
> 6. 神经网络API 1.1

### 11. [热修复原理](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fcsdn_lqr%2Farticle%2Fdetails%2F78534065)

热修复的原理是让我们的新类替换掉原来类的加载，从而达到修复的目的，以下是一种思路：

> 1. java中通过PathClassLoader和DexClassLoader来加载类，类加载的方式是双亲委派模式
> 2. PathClassLoader和DexClassLoader都继承自BaseDexClassLoader
> 3. BaseDexClassLoader中维护了一个dex的数组
> 4. 我们可以通过DexClassLoader加载类，然后通过反射的机制将加载进来的数组添加到path数组的前面
> 5. 加载的时候找到我们需要的class后，就不再继续向后找了，所以可以达到修复的目的

### 12. Android中的进程间通信（IPC）

> 1. Bundle : 只支持四大组件
> 2. 文件共享：不适合并发
> 3. Messenger：封装了AIDL
> 4. AIDL：通过binder实现
> 5. ContentProvider：共享数据
> 6. Socket：适用于网络等

### 13. [解决Android7.0 更新安装包时不能自动安装问题](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc58aa241688c)

> 1. Android 7.0中私有目录访问会被限制，导致不能自动安装
> 2. 可以使用FileProvider来解决

### [如何开启多进程](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F11da30127823)?应用是否可以开启N个进程？

> 1. 通过在AndroidManifest中给Activity设置process属性开启新的进程
> 2. 可以开启N个进程，例如给webview单独开启一个进程，但要处理多进程间通信和多次初始化Handler问题

### [Service先start再bind如何关闭service](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fee224f18a4bd)

> 先unbind,再stop

### 为什么bindService可以跟Activity生命周期联动

> 1. 在Activity退出时调用unbind方法，service会销毁
> 2. 如果不调用unbind方法，service也会销毁，但是[会抛出leaked serviceConnection 异常](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fvokie%2Farchive%2F2013%2F04%2F15%2F3602088.html) ([参考2](https://link.juejin.im/?target=https%3A%2F%2Fyiweifen.com%2Fv-1-335376.html))

### [子线程中如何使用Handler](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Flang-yu%2Fp%2F6228832.html)

> 1. 使用HandlerThread,新建Handler时通过调用HandlerThread 的 getLooper方法拿到looper
> 2. 原理：HandlerThread在run时会为我们生成一个looper，getLooper方法会阻塞等待直到 looper！=null 才返回。

### 如何进行单元测试

> 1. [Junit](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F79addb29b06d):不需要依赖android环境，适合于逻辑测试
> 2. [Instrumentation](https://link.juejin.im/?target=https%3A%2F%2Fwww.oschina.net%2Fquestion%2F54100_27061):依赖android环境，可以启动Activity，模拟内存回收，获取组件等，模拟点击等。需要在AndroidManifest中进行配置，适合于更复杂的测试

### [TabLayout如何设置指示器的宽度](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fqq_34664695%2Farticle%2Fdetails%2F78535572)

> 通过反射拿到对应的指示器，设置LayoutParams

### Android中如何查看一个对象的回收情况

> 1. 外部：通过adb shell 命令导出内存，借助工具分析
> 2. 内部：通过将对象加入WeakReference，配合RefernceQueue观察对象是否被回收，被回收的对象会被加入到RefernceQueue中

### [回形打印二维数组](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F35e618aec35f)

> 思路：递归实现，分别打印每一圈

### APK内容



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="434" height="163"></svg>)



### [class文件如何转化成dex](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F2eb518941681)

> 使用build tools 下的dx工具

```
class 和dex文件对比:
1. 都是二进制文件
2. class文件存在容与，dex文件将整个工程的类信息整合到了一起，去掉了冗余
复制代码
```

### [硬件加速](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F40f660e17a73)

> 1. 随便写点凑个数吧=-=
> 2. 硬件加速的四个级别：Application/Activity/Window/View

### [为什么选择Binder作为通信方式](https://link.juejin.im/?target=https%3A%2F%2Fblog.csdn.net%2Fstarter110%2Farticle%2Fdetails%2F49616701)

> 1. binder效率更高：socket是一个通用接口，效率低；管道和队列内存拷贝两次，效率低；共享内存控制复杂
> 2. binder更加安全：binder可以建立私有通道，通过uid/pid验证身份

## 参考资料

[LearningNotes](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffrancistao%2FLearningNotes)

[40 个 Android 面试题](https://juejin.im/entry/57aad0728ac247005f4cfa81)

https://www.nowcoder.com/discuss/3043

http://weixin.niurenqushi.com/article/2017-03-17/4790406.html

https://blog.csdn.net/vfush/article/details/51481127

https://blog.csdn.net/vfush/article/details/51790079