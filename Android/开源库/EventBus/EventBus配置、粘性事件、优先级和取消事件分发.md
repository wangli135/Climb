# EventBus配置  
前面的博客中获取EventBus，都是使用EventBus.getDefault()，而如果需要对EventBus进行配置，那么需要使用EventBus.Builder进行设置。  
```
EventBus eventBus = EventBus.builder()
    .logNoSubscriberMessages(false)
    .sendNoSubscriberEvent(false)
    .build();
```
# 粘性事件  
一般情况下，发布者将事件发出，如果没有对该事件感兴趣的订阅者，那么这条消息就消失了；而粘性事件则允许，在订阅者后来注册到事件中心，还能收到该事件。  
在[EventBus简介以及初步使用](https://blog.csdn.net/qq_19431333/article/details/80976445)中，Activity A是订阅者，Activity B是发布者，这儿我们换一下，Activity A发布一个粘性事件，代码如下：  
```
public void downloadPage(View view) {
        EventBus.getDefault().postSticky(new DownloadEvent());
        startActivity(new Intent(this,SecondActivity.class));
    }
```
Activity B的代码如下：  
```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        EventBus.getDefault().register(this);
    }

    @Subscribe(sticky = true)
    public void download(DownloadEvent downloadEvent) {
        Toast.makeText(this,"Receive Sticik Event",Toast.LENGTH_LONG).show();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }
```
当Activity A跳转到Activity B时将会有Toast提示；当把Activity B中download方法的注解stick修改为false后，将不再有Toast提示，从而可以看到粘性事件是如何作用的。  
## 手动获取和取消粘性事件  
如果需要手动取消粘性事件，那么可以执行以下代码：  
```
DownloadEvent stickyEvent = EventBus.getDefault().getStickyEvent(DownloadEvent.class);
        if(stickyEvent!=null){
            EventBus.getDefault().removeStickyEvent(stickyEvent);
        }

        DownloadEvent previousEvent = EventBus.getDefault().removeStickyEvent(DownloadEvent.class);
        if(previousEvent!=null){
            EventBus.getDefault().removeStickyEvent(previousEvent);
        }

```
# 优先级  
Subscribe注解可以使用priority进行修饰，同一分派线程中，优先级越高，订阅方法越先处理。  
## 取消事件分发  
```
@Subscribe
public void onEvent(MessageEvent event){
    EventBus.getDefault().cancelEventDelivery(event) ;
}
```
一旦取消了事件的分发，事件将不再继续向下进行分发。
一点设想，**可以通过优先级+取消事件分发构建拦截链**。