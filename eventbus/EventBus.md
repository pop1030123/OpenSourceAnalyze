####EventBus源码分析笔记

事件总线，类似的实现还有广播,square公司的otto库；整体的实现模式是使用观察者模式，整个库代码量不多，2700行左右；
#####1.EventBus类
EventBus.java是整个库代码行数最多的类，不过代码行数不超过500行，也是最重要的类,它对外提供了一系列方法，基本上所有的操作都是使用这个类来完成的。<br>
　　首先，要使用EventBus，必须创建EventBus对象，EventBus库提供一个默认的单例实现以方便使用；

~~~java
/** Convenience singleton for apps using a process-wide EventBus instance. */
    public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
~~~
EventBus类的一些主要的方法如下：

| 方法名 | 描述 |
|:--|:--:|
|public static EventBus getDefault()|EventBus默认单例对象，方便APP全局使用|
|public void register(Object subscriber)|注册需要观察的对象，被观察的对象改变后，所有注册的对象都会收到事件，2.4版本需要特别调用registerSticky()|
|public synchronized void unregister(Object subscriber)|解注册需要观察的对象|
|public synchronized boolean isRegistered(Object subscriber)|判断对象是否已经被注册，防止重复注册的异常出现|
|public void post(Object event)|发布事件，凡是注册了event的观察者对象都会收到消息|
|public void postSticky(Object event)|发布sticky事件，即事件发布后，再注册，也可以收到发布的事件|
|public boolean removeStickyEvent(Object event)|取消sticky事件|
|public void removeAllStickyEvents()|取消所有的sticky事件|
#####2.EventBus时序图
<img src="sequence.png" width=400 height=437/>
#####3.EventBus单一事件接收流程图
<img src="single-flow.png" width=442 height=400/>
#####4.自定义EventBus对象 todo
#####5.线程模式ThreadMode
发布事件的任务和处理事件的任务可以在不同的线程中执行，比如发布事件的任务是在子线程中，而处理事件的任务是在UI线程中，所以需要用线程模式来处理不同的需求；

| 线程模式 | 描述 |
|:--|:--:|
|POSTING|发布线程和处理线程是同一个线程，这样速度是最快的，可以避免线程切换的消耗，EventBus默认是这样处理的。|
| MAIN |处理线程是在UI线程中。如果发布任务的线程是在子线程中，那么会切换线程进行处理，如果是在主线程中，则直接调用；|
| BACKGROUND |在子线程中处理事件；如果发布线程是在非主线程，那么会直接调用处理事件；如果是在主线程，那么会在单独的子线程中依次处理事件；|
| ASYNC |异步处理；不同的订阅者会在不同的线程中处理事件；一般处理一些比较耗时的任务，比如访问网络，或者大数据运算等；EventBus会使用线程池来处理不同的事件；|