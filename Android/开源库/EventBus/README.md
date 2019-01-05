EventBus是一个事件总线模型，可以解决组件之间通信的问题。期间学习EventBus，有一些关于发布订阅模型的思考。  

# 事件  
1. 事件如何定义，包含各种需求

    EventBus中的事件是Object，具体需求自己添加    

2. 高级特性,事件的参数配置
# 发布者  
1. 发布者如何将事件发出，不需要关心事件的最终流向
  发布者需要做两件事，一是创建事件，二是将事件发给事件中心
```
MessageEvent messageEvent=new MessageEvent();
                messageEvent.setMessage("Hello World From Network");
                EventBus.getDefault().post(messageEvent);
```
# 订阅者  
1. 订阅者如何将自己注册到事件中心，以让注册中心能够找到自己
```
EventBus.getDefault().register(this);
```
订阅者将自己注册给事件中心；
2. 订阅者需要知道怎么处理订阅的东西
```
@Subscribe
    public void showContent(MessageEvent messageEvent){
//        showContentTv.setText(messageEvent.getMessage());
        Log.i("TAG",Thread.currentThread().getName()+"  "+messageEvent.getMessage());
    }
```
@Subscribe注解说明该方法是订阅者中的具体处理订阅事件的逻辑，该方法的参数是其感兴趣的参数。
# 事件中心  
1. 怎么保存注册者注册者、注册的东西以及订阅者  
2. 如何将正确的事件分发给恰当的订阅者  
3. 高级特性，线程调度、粘性事件
4. 思考改进点：事件拦截，事件多处理是否支持？  

