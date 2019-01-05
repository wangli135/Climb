EventBus支持线程分发，在上一篇博客[EventBus简介以及初步使用](https://blog.csdn.net/qq_19431333/article/details/80976445)中，了解到EventBus的使用主要涉及事件发送者，以及事件订阅者；对于发送和订阅这两个行为，可以在不同的线程中，这就是EventBus的线程分发。关于线程的设置，可以在订阅方法中使用@Subscribe注解进行线程的调节，如代码所示：  
```
@Subscribe(threadMode = ThreadMode.MAIN)
    public void showContent(MessageEvent messageEvent){
        showContentTv.setText(messageEvent.getMessage());
    }
```
上面代码，订阅的这个方法将在主线程中执行。
ThreadMode一共有五种值：分别是：  
- ThreadMode.POSTING 
- ThreadMode.MAIN
- ThreadMode.MAIN_ORDERED
- ThreadMode.BACKGROUND
- ThreadMode.ASYNC  

### ThreadMode.POSTING  
这是默认的一种策略，订阅处理的线程和事件产生位于同一线程中。这种模型是开销最小的，因为不需要线程切换。因此建议这种模型下，执行不需要UI主线程的简单任务。
下面仍以上一篇博客的例子介绍，当把订阅方法改为如下时:  
```
@Subscribe()
    public void showContent(MessageEvent messageEvent){
        showContentTv.setText(messageEvent.getMessage());
    }
```
再运行程序，Activty A将不会显示Activity B传递的内容，并且可以在logcat中看到以下异常信息：  
```
Could not dispatch event: class com.example.wangli.eventbusdemo.MessageEvent to subscribing class class com.example.wangli.eventbusdemo.MainActivity
    android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views
```
可以发现这个异常就是说明不能在非UI线程操作UI。上面的注解等同如下：  
```
@Subscribe(threadMode=ThreadMode.POSTING)
```
### ThreadMode.MAIN
在Android平台，订阅方法将会在UI线程中被调用，如果事件产生是在主线程中，那么处理也会直接在主线程调用，这个会阻塞事件产生，因此方法处理需要耗时较少；否则就会进入主线程处理的队列。如果在非Android平台，那和POSTING一样。  
举个例子，将Actiity A的订阅方法修改如下：  
```
@Subscribe(threadMode = ThreadMode.MAIN)
    public void showContent(MessageEvent messageEvent){
        showContentTv.setText(messageEvent.getMessage());
        Log.i("TAG","Activity A");
    }
```
#### 事件产生在主线程  
当事件产生在主线程，事件处理将会阻塞事件产生，Activity B的代码如下：  
```
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        MessageEvent messageEvent = new MessageEvent();
        messageEvent.setMessage("Hello World From Network");
        EventBus.getDefault().post(messageEvent);
        Log.i("TAG", "Activity B");
        finish();

    }
```
这种情况下的运行日志是：  
```
Activity A  
Activity B
```
因为阻塞了产生事件，所以先打印了Activity A中的日志，而后再打印Activity B中的日志。  
#### 事件产生在非主线程  
当事件产生不在主线程时，将会进入队列，相对是一种异步行为，将Activity B的代码改成如下：  
```
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        new Thread(new Runnable() {
            @Override
            public void run() {

                //模拟网络任务
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                MessageEvent messageEvent=new MessageEvent();
                messageEvent.setMessage("Hello World From Network");
                EventBus.getDefault().post(messageEvent);
                Log.i("TAG","Activity B");
                finish();

            }
        }).start();

    }
```
运行结果如下：  
```
Activity B
Activity A
```
### ThreadMode.MAIN_ORDERED  
在Android平台中，处理将会在UI线程中调用。不同于MAIN，总是会被分发到主线程的队列中，不会阻塞post线程。  
将Activity A事件处理改成如下：  
```
@Subscribe(threadMode = ThreadMode.MAIN_ORDERED)
    public void showContent(MessageEvent messageEvent){
        showContentTv.setText(messageEvent.getMessage());
        Log.i("TAG","Activity A");
    }
```
将Activity B post线程也改成在主线程中那段，运行结果如下：  
```
Activity B
Activity A
```
由于异步了，所以和MAIN的情形不一样了。  
### ThreadMode.BACKGROUND
在Android平台中，事件处理会在background线程中调用。如果post不是在主线程，那么事件处理会被直接在post线程中调用；如果post是主线程，EventBus使用了一个单一的background线程，那么所有主线程post的事件将会按照队列顺序进入，因此这要求事件处理尽可能快速返回，不能阻塞background线程。如果不是在android平台中，那么总是会使用一个background线程。  
将Activity A中的代码改成： 
```
@Subscribe(threadMode = ThreadMode.BACKGROUND)
    public void showContent(MessageEvent messageEvent){
//        showContentTv.setText(messageEvent.getMessage());
        Log.i("TAG",Thread.currentThread().getName()+"  "+messageEvent.getMessage());
    }
```
将Activity B中的代码改成如下:  
```
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        new Thread(new Runnable() {
            @Override
            public void run() {

                //模拟网络任务
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                MessageEvent messageEvent=new MessageEvent();
                messageEvent.setMessage("Hello World From Network");
                EventBus.getDefault().post(messageEvent);
                Log.i("TAG",Thread.currentThread().getName());
                finish();

            }
        }).start();

        MessageEvent messageEvent = new MessageEvent();
        messageEvent.setMessage("Hello World From Network");
        EventBus.getDefault().post(messageEvent);
        Log.i("TAG-Main", Thread.currentThread().getName());


        messageEvent.setMessage("Hello");
        EventBus.getDefault().post(messageEvent);
        Log.i("TAG-Main", Thread.currentThread().getName());


        finish();


    }
```
最终的运行结果如下：  
```
TAG-Main: main
TAG-Main: main
TAG: pool-1-thread-1  Hello
TAG: pool-1-thread-1  Hello
TAG: Thread-210  Hello World From Network
TAG: Thread-210
```
可以发现几个问题，post是主线程，background线程始终是同一个；post不是主线程，background与post相同，且事件处理是会阻塞post线程的。
### ThreadMode.ASYNC
事件处理总是在一个单独的线程。总是与post线程和main线程独立。如果操作耗时，比如网络操作，或者大量运算，那么应该使用这种模式，EventBus后台使用线程池管理这些线程。  
继上面的例子，将事件处理改成ASYNC：  
```
@Subscribe(threadMode = ThreadMode.ASYNC)
    public void showContent(MessageEvent messageEvent){
//        showContentTv.setText(messageEvent.getMessage());
        Log.i("TAG",Thread.currentThread().getName()+"  "+messageEvent.getMessage());
    }
```
Activity B中的post逻辑不变，结果如下：  
```
TAG-Main: main
TAG: pool-1-thread-1  Hello
TAG-Main: main
TAG: pool-1-thread-2  Hello
TAG: Thread-215
TAG: pool-1-thread-2  Hello World From Network
```
可以发现，事件处理总是在线程池的线程中。  
### 总结  
经过上面的分析，可以知道每种ThreadMode的使用场景以及与post线程不同时，有怎样的表现。  

| 发布线程     | Android主线程                   | 非Android主线程，线程a                             |
| ------------ | ------------------------------- | -------------------------------------------------- |
| POSTING      | Android主线程                   | 非Android线程，线程a                               |
| MAIN         | Android主线程，阻塞主线程的发布 | 进入主线程的队列                                   |
| MAIN_ORDERED | 主线程队列                      | Android平台会进入主线程队列，Java平台与POSTING一样 |
| BACKGROUND   | background线程                  | 非Android主线程，线程a                             |
| ASYNC        | 单独线程c                       | 单独线程c                                          |
表格中，表头表示发布所处的线程，订阅方法处于不同ThreadMode，订阅方法将在哪个线程中执行。  