### EventBus简介  
[EventBus](https://github.com/greenrobot/EventBus)是一个Android和Java的发布/订阅事件总线模型，其模型图如下所示：
![EventBus模型](https://github.com/greenrobot/EventBus/raw/master/EventBus-Publish-Subscribe.png)

EventBus的优点有：  
1. 简化组件之间的通信  
2. 简化代码
3. 快速，小巧（~50K jar）
4. 具有线程分发、订阅优先级等高级特性  

从模型图中可以看到几个概念：  
1. Publisher：发布者，发布事件，事件的产生者  
2. Event：事件，封装了要传递的内容  
3. Subscriber：订阅者，对事件感兴趣的消费者  
4. EventBus：事件总线，管家，将事件分发给正确的Subscriber进行处理  

### 初步使用  
在Android中考虑一个场景，Activity A需要显示一个从网站上下载的内容，该下载任务在Activity B中进行，完成后，将内容通知给Activity A进行显示。  
#### 不使用EventBus的情况
不使用EventBus的情况下，Activity A使用startActivityForResult请求Activity B，Activity中开启线程进行网络任务，完成后将结果通过setResult返回。  
Actiivty A的代码如下：  
```
private TextView showContentTv;

public void downloadPage(View view) {
        startActivityForResult(new Intent(this,SecondActivity.class),0);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        showContentTv.setText(data.getStringExtra("content"));
    }
```
Activity B的代码如下：  
```
private Handler handler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            if(msg.what==1){
                Intent intent=getIntent();
                intent.putExtra("content","Hello World From Network");
                setResult(0,intent);
                finish();
            }
        }
    };

    @Override
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
                handler.sendEmptyMessage(1);

            }
        }).start();

    }
```
#### 使用EventBus的情况
EventBus的基本使用包含三个步骤：  
1. 定义事件  
  例子中下载内容就是一个String类型的字符串，可以简单定义：
```
public class MessageEvent {

    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```
2. 准备订阅者  
  由于上面例子中Activity A是要显示Activity B下载的内容，因此Activity A是事件的消费者，在其中声明一个方法如下：  
```
@Subscribe(threadMode = ThreadMode.MAIN)
    public void showContent(MessageEvent messageEvent){
        showContentTv.setText(messageEvent.getMessage());
    }
```
另外，还需要将Subscriber注册到EventBus中，这个需要在Activity的生命周期方法中进行注册，如下：  
```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        showContentTv=findViewById(R.id.showContentTv);
        EventBus.getDefault().register(this);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }
```
3. 发布者发布事件
  Activity B完成下载后，需要将内容打包成Event，然后发送出去，  
```
 @Override
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
                finish();

            }
        }).start();
    }
```
### 总结  
经过两种方式的对比，可以看到接入EventBus是一件很简单的事情，并且EventBus可以带来很好的解耦性。