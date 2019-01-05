 [EventBus的线程分发](https://blog.csdn.net/qq_19431333/article/details/80992088)中介绍了EventBus中发布和订阅方法设置了ThreadMode之间的关系，最终表格如下：
| 发布线程     | Android主线程                   | 非Android主线程，线程a                             |
| ------------ | ------------------------------- | -------------------------------------------------- |
| POSTING      | Android主线程                   | 非Android线程，线程a                               |
| MAIN         | Android主线程，阻塞主线程的发布 | 进入主线程的队列                                   |
| MAIN_ORDERED | 主线程队列                      | Android平台会进入主线程队列，Java平台与POSTING一样 |
| BACKGROUND   | background线程                  | 非Android主线程，线程a                             |
| ASYNC        | 单独线程c                       | 单独线程c                                          |
表格中，表头表示发布所处的线程，订阅方法处于不同ThreadMode，订阅方法将在哪个线程中执行。  
表格中，表头表示发布所处的线程，订阅方法处于不同ThreadMode，订阅方法将在哪个线程中执行。  
在[EventBus源码分析之订阅-发布模型](http://blog.csdn.net/qq_19431333/article/details/81226529)中已经介绍了EventBus的发布-订阅模型的源码，最终通过反射调用了订阅者的方法，而根据订阅方法@Subscribe注解中ThreadMode设置，订阅方法的调用将在不同的线程中被执行。本篇博客将从线程分发的角度分析EventBus的源码。  
订阅处理最终会走到下面这个方法，  
```
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
```
下面将根据ThreadMode的不同类型进行分析。  
## ThreadMode=POSTING  
ThreadMode为POSTING的情况，将调用invokeSubscriber()方法，该方法的代码如下：  
```
void invokeSubscriber(Subscription subscription, Object event) {
        try {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }
```
从代码可以看出，在发布者post()之后，post()方法的内部将会在同一个线程中反射调用订阅者的订阅方法进行消费，所以ThreadMode是POSTING的情况下，**发布与订阅是在同一个线程中的，并且订阅是紧接着发布的**。  
## ThreadMode=MAIN  
ThreadMode为MAIN的情况，对事件发布的线程进行了区分：  
```
case MAIN:      
                //case:主线程
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                }
                //case:非主线程
                else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
```
代码可以看到，
- 如果发布者是在主线程中post事件的，那么处理逻辑和POSTING一样，由于是同步的，所以会阻塞发布者；  
- 如果发布者是在非主线程中post事件的，那么将事件入列，下面具体分析这种情况。  

这里mainThreadPoster对象是Poster接口，实现类HandlerPoster的声明如下：  
```
public class HandlerPoster extends Handler implements Poster
```
可以看到HandlerPoster其实是一个Handler，并且其Looper=Looper.getMainLooper()，因此该Handler就是接受主线程中发出的消息的。  
### HandlerPoster.enqueue()  
```
public void enqueue(Subscription subscription, Object event) {
        //创建PendingPost对象
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        synchronized (this) {
            //加入队列
            queue.enqueue(pendingPost);
            //如果不在处理中，那么发送一个Message激活激活处理流程
            if (!handlerActive) {
                handlerActive = true;
                //发送一个Message
                if (!sendMessage(obtainMessage())) {
                    throw new EventBusException("Could not send handler message");
                }
            }
        }
    }
```
从上面的代码可以看到，enqueue()操作就是讲Subscription和Object组合成PengingPost对象，然后加入到队列中，然后发送的是一个Message对象；发送完Message,Handler的接受逻辑是在handeMessage()方法中。  
**发布是在非主线程中的，因此enqueue()方法就是在非主线程中，因为同时可能会有多个线程发布消息，因此这里进行了同步。PendingPostQueue是一个链表实现的队列。**
### HandlerPoster.handleMessage()  
```
@Override
    public void handleMessage(Message msg) {
        boolean rescheduled = false;
        try {
            long started = SystemClock.uptimeMillis();
            while (true) {
                //从队列中取PengingPost
                PendingPost pendingPost = queue.poll();
                //双重检查，由于enqueue存在并发问题
                if (pendingPost == null) {
                    synchronized (this) {
                        // Check again, this time in synchronized
                        pendingPost = queue.poll();
                        //没有要处理的事件了，置标志位为false，表示不在处理中了
                        if (pendingPost == null) {
                            handlerActive = false;
                            return;
                        }
                    }
                }
                //反射调用订阅方法
                eventBus.invokeSubscriber(pendingPost);
                //记录进入while循环的事件
                long timeInMethod = SystemClock.uptimeMillis() - started;
                //处理超时，默认10ms，跳出，但是处理中激活标志仍为true，因为又发了一条Message
                if (timeInMethod >= maxMillisInsideHandleMessage) {
                    if (!sendMessage(obtainMessage())) {
                        throw new EventBusException("Could not send handler message");
                    }
                    rescheduled = true;
                    return;
                }
            }
        } finally {
            handlerActive = rescheduled;
        }
    }
```
从上面两段代码可以看出，enqueue()方法并不会每次都发送Message激活handleMessage()，这是通过handlerActivi标志位进行控制的。那么enqueue()中那些没有被消费的事件该怎么消费呢？答案是handleMessage()中的while死循环，但是为了避免一直在死循环中处理事件影响主线程的性能，又设置了一个超时时间，一旦执行了超时了，那么再发送一个Message并且退出，那么Handler的机制可以保证过会儿又能进入到handleMessage()方法中继续处理队列中的事件。  
核心还是通过反射进行调用的，这儿也能看出订阅方法的执行是在主线程中的。但是由于enqueue()的存在，订阅与发布是异步的，订阅的消费不会阻塞发布。   
## ThreadMode=MAIN_ORDERED  
ThreadMode为MAIN_ORDERED的情况下，首先进行了判断：  
```
 case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
```
如果mainThreadPoster不为null，那么执行的逻辑和ThreadMode=MAIN时，在非主线程中post事件一样，执行异步操作；否则执行的逻辑和ThreadMode=POSTING一样。那么重点来分析一下mainThreadPoster在什么情况下会为null。  
### EventBusBuilder
mainThreadPoster可以在EventBusBuilder中进行配置，EventBus是通过Builder模式创建的，EventBus.getDefault()使用的就是默认的Builder，而如果我们想对EventBus进行配置，可以使用EventBusBuilder。  
```
EventBus(EventBusBuilder builder) {
        ...
        mainThreadSupport = builder.getMainThreadSupport();
        mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
        ...
    }
```
从EventBus的构造方法中可以看出mainThreadPoster和mainThreadSuppoert有关，而该对象表示当前程序平台是否支持主线程，Builder中的定义如下：  
```
MainThreadSupport getMainThreadSupport() {
        //case：不为null，直接返回
        if (mainThreadSupport != null) {
            return mainThreadSupport;
        }
        //case：Android平台，返回AndroidHandlerMainThreadSuppoet，传入的对象是MainLooper
        else if (Logger.AndroidLogger.isAndroidLogAvailable()) {
            Object looperOrNull = getAndroidMainLooperOrNull();
            return looperOrNull == null ? null :
                    new MainThreadSupport.AndroidHandlerMainThreadSupport((Looper) looperOrNull);
        }
        //case：非Android平台，null
        else {
            return null;
        }
    }
```
需要注意的是EventBusBuilder关于mainThreadSupport参数是个只读参数，不可以进行配置；EventBus对Android平台做了特殊处理，在Android平台上存在UI(主)线程和非UI(主)线程的区分。而判断是否是Android平台的方法是反射android.log.Log类，成功了表示是Android平台，失败了表示不是Android平台。  

因此可以得出结论，订阅方法使用了MAIN_ORDERED注解后，对应的情况有两种：  
- 如果在Android平台上，那么事件将异步地在Android 主线程中执行；
- 如果在非Android平台上，那么事件将同步地在发布线程中执行。  

## ThreadMode=BACKGROUND  
当ThreadMode为BACKGROUND的情况下，对发布的线程进行了区分：  
```
case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
```
- 如果发布事件的线程是在主线程中的，那么将事件加入到BackgroundPoster的队列中；
- 如果发布事件的线程是在非主线程中，那么执行逻辑和POSTING一样，同步地在发布线程中被消费。  

BackgroundPoster是一个实现了Runnable和Poster接口的类，其定义如下：  
```
final class BackgroundPoster implements Runnable, Poster {

    private final PendingPostQueue queue;
    private final EventBus eventBus;

    private volatile boolean executorRunning;

    ...

    public void enqueue(Subscription subscription, Object event) {
        //创建PendingPost对象
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        synchronized (this) {
            //入队
            queue.enqueue(pendingPost);
            if (!executorRunning) {
                executorRunning = true;
                //交给EventBus的线程池，执行自己
                eventBus.getExecutorService().execute(this);
            }
        }
    }

    @Override
    public void run() {
        try {
            try {
                while (true) {
                    //获取PengingPost
                    PendingPost pendingPost = queue.poll(1000);
                    if (pendingPost == null) {
                        synchronized (this) {
                            // Check again, this time in synchronized
                            pendingPost = queue.poll();
                            if (pendingPost == null) {
                                executorRunning = false;
                                return;
                            }
                        }
                    }
                    //反射调用
                    eventBus.invokeSubscriber(pendingPost);
                }
            } catch (InterruptedException e) {
                eventBus.getLogger().log(Level.WARNING, Thread.currentThread().getName() + " was interruppted", e);
            }
        } finally {
            executorRunning = false;
        }
    }

}
```
可以发现，BackgroundPoster的逻辑和主线程的HandlerPoster类似，区别在于，HandlerPoster中的invokeSubscriber()方法是在主线程中调用的，而BackgroundPoster中的invokeSubscriber()是在线程池中，即非UI线程中执行的；具体用到的控制标志位和HandlerPoster中一样。  
## ThreadMode=ASYNC  
ThreadMode为ASYNC的情况下，就不再进行区分了，直接交给AsyncPoster进行入对：  
```
 case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
```
AsyncPoster的源代码如下：  
```
class AsyncPoster implements Runnable, Poster {

    private final PendingPostQueue queue;
    private final EventBus eventBus;

    ...
    
    public void enqueue(Subscription subscription, Object event) {
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        queue.enqueue(pendingPost);
        eventBus.getExecutorService().execute(this);
    }

    @Override
    public void run() {
        PendingPost pendingPost = queue.poll();
        if(pendingPost == null) {
            throw new IllegalStateException("No pending post available");
        }
        eventBus.invokeSubscriber(pendingPost);
    }

}
```
可以发现与HandlerPoster、BackgroundPoster代码都是很相似的；区别主要有两点：  
- 每发布一个事件，AsyncPoster的run()方法都会被执行一次；而BackgroundPoster的run()和HandlerPoster的handleMessage()不会每次都执行，它们辅助通过队列以及标志位进行了控制以及实现；
- AsyncPoster中没有进行同步加锁。

可以看到与Background的区别，ASYNC模型下，总是会创建一个新的线程进行调用invokeSubscriber()方法的；而Background中有个死循环，会存在很多情况下，很多POST的处理是在同一个线程中的。  
### EventBus的线程池  
不论ThreadMode是BACKGROUND还是ASYNC的情况下，都涉及到了EventBus的线程池设置，getExecutorService()方法如下：  
```
ExecutorService getExecutorService() {
        return executorService;
    }
```
最终默认情况下使用的是EventBusBuilder中的线程池：  
```
 private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();
```
惊奇地发现，EventBus竟然使用了CachedThreadPool线程池，好吧，那看来如果我们存在很多BACKGROUND和ASYNC的情况下，还是需要自己配个线程池的。  
## 总结  
经过对线程分发部分代码的分析，可以看到会在四处地方调用invokeSubscribe()方法进行具体的订阅方法执行，分别是：  
1. 与post在同一线程中，同步调用；
2. 在HandlerPoster的handleMessage(),这种情况下是在主线程中异步地执行的；
3. 在BackgroundPoster的run()，这种情况下是异步执行地，大多数情况下订阅方法在同一个线程中同步被调用；
4. 在AsyncPoster的run()，这种情况是异步执行第，并且订阅方法在不同的线程中被调用。  