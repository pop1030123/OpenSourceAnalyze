####Picasso源码分析
#####1.最简单的使用方式；

~~~java
Picasso.with(activity)
.load(mImageUrl)
.resizeDimen(100,200)
.into(mTargetView);
~~~
#####2.图片加载线程池**PicassoExecutorService**；

核心|最大|存活时间|单位|队列
:-:|:-:|:-:|---|----
&nbsp;&nbsp;&nbsp;3&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;3&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|MILLISECONDS|PriorityBlockingQueue

#####3.根据网络状态自动调整线程池的大小；
  
  网络状态|线程数量
  :---|:-:
  wifi/winmax/ethernet|4
  (4G)LTE/hspap/ehrpd|3
  (3G)umts/cdma/evdo_(0/a/b)|2
  (2G)gprs/edge|1
  其它|3
  
  通过监听网络变化的广播(ConnectivityManager.CONNECTIVITY_ACTION)来达到调整线程池的目的。具体设置的方法为：
  
  ~~~java
  com.squareup.picasso.PicassoExecutorService
  void adjustThreadCount(NetworkInfo info)
  ~~~
  在Picasso对象创建的时候，注册监听了BroadcastReceiver。
#####4.Picasso使用了单例模式；

~~~java
public static Picasso with(@NonNull Context context) {
    if (context == null) {
      throw new IllegalArgumentException("context == null");
    }
    if (singleton == null) {
      synchronized (Picasso.class) {
        if (singleton == null) {
          singleton = new Builder(context).build();
        }
      }
    }
    return singleton;
  }
~~~
每一次调用加载图片的时候，都校验了单例对象；**Builder**类专门用来创建Picasso这个单例对象；
#####5.Picasso默认创建时的参数；

  下载器|缓存机制|执行器|转换器(Request)
  :---|:-:|:--|:--
    API>=9:okhttp3.OkHttpClient<br>API<9:com.squareup.okhttp.OkHttpClient| LruCache | PicassoExecutorService |RequestTransformer.IDENTITY

#####6.自定义参数；

