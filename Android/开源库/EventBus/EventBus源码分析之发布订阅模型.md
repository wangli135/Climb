EventBus事件总线模式如下图：  
![EventBus模型](https://github.com/greenrobot/EventBus/raw/master/EventBus-Publish-Subscribe.png)
本文主要从两个方面介绍源码：  
1. 订阅者是如何注册到事件中心的；
2. 发布者发布了事件之后，事件中心是如何将事件调用到合适的订阅方法的。  

# 订阅者注册到事件中心  
订阅者注册到事件中心需要调用如下代码： 
```
EventBus.gtetDefault().register(this)
```
订阅者可以是任何对象，唯一的要求是内部有@Subscribe修饰的方法，该方法是有一定要求的，这可以在后面的源码中看到EventBus对该方法的要求。   
## EventBus.getDefault()  
该方法的代码如下：  
```
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
```
可以看到EventBus是一个单例模式，也很好理解，毕竟管理者所有的订阅和和事件，有且只能有一个，单例决定了EventBus只能用于单个进程中的发布-订阅。  
## EventBus.register()
register()方法的参数是Object，说明任何对象都可以是订阅者，该方法源码如下：  
```
public void register(Object subscriber) {
        Class<?> subscriberClass = subscriber.getClass();
        //找到该类中所有被@Subscribe注解的合法方法
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            //将对象、方法作为元组注册到事件中心
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```
从代码中可以看出主要有两步：  
1. 找到@Subscribe修饰的方法，以SubscriberMethod对象表示方法的信息；
2. 将对象、方法作为元组注册到事件中心。
  下面首先看一下SubscriberMethod的定义:
### SubscriberMethod类的定义  
```
public class SubscriberMethod {
    final Method method;
    final ThreadMode threadMode;
    final Class<?> eventType;
    final int priority;
    final boolean sticky;
    /** Used for efficient comparison */
    String methodString;
}
```
上面就是其主要字段，一个合格的订阅方法（以Method对象表示）+@Subscribe注解中的信息：ThreadMode、EventType、priority、sticky。methodString字段是用来比较两个对象SubscriberMethod对象是否相等的依据。  
### SubScriberMethodFinder.findSubscriberMethods()  
SubscriberMethodFind是用来找@Subscribe注解了方法的工具类，findSubscriberMethosd()的定义如下：  
```
private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();

List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
//从缓存中查找该订阅者中的订阅方法
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        //找到了，直接返回
        if (subscriberMethods != null) {
            return subscriberMethods;
        }

        
        if (ignoreGeneratedIndex) {
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        //如果一个合格的订阅方法都没找到，抛出异常
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        }
        //存入缓存并返回结果
        else {
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }
```
如果使用默认的EventBus，那么ignoreGeneratedIndex为false；由于查找订阅方法是一个耗时操作，因此SubscriberMethodFinder这儿对方法列表进行了缓存；这里先暂时不管该参数，那么僵使用findUsingReflection()方法进行查找，该方法的定义如下：  
### SubscriberMethod.findUsingReflection()  
```
private static final int POOL_SIZE = 4;
    private static final FindState[] FIND_STATE_POOL = new FindState[POOL_SIZE];

private List<SubscriberMethod> findUsingReflection(Class<?> subscriberClass) {
        //获取FindState对象
        FindState findState = prepareFindState();
        //初始化
        findState.initForSubscriber(subscriberClass);
        //从该类开始向上寻找订阅方法
        while (findState.clazz != null) {
            findUsingReflectionInSingleClass(findState);
            findState.moveToSuperclass();
        }
        //获取FindState中找到的订阅方法
        return getMethodsAndRelease(findState);
    }

private FindState prepareFindState() {
        synchronized (FIND_STATE_POOL) {
            for (int i = 0; i < POOL_SIZE; i++) {
                FindState state = FIND_STATE_POOL[i];
                if (state != null) {
                    FIND_STATE_POOL[i] = null;
                    return state;
                }
            }
        }
        return new FindState();
    }
    
```
这里可以看到，对FindState对象做了一个容量为4的对象池。由此可以推断FindState是一个相对重量级的对象，所以才做了对象池，其定义如下：  
#### FindState类定义  
FindState是SubscriberMethodFinder的静态内部类：
```
static class FindState {
        final List<SubscriberMethod> subscriberMethods = new ArrayList<>();
        final Map<Class, Object> anyMethodByEventType = new HashMap<>();
        final Map<String, Class> subscriberClassByMethodKey = new HashMap<>();
        final StringBuilder methodKeyBuilder = new StringBuilder(128);

        Class<?> subscriberClass;
        Class<?> clazz;
        boolean skipSuperClasses;
        SubscriberInfo subscriberInfo;

        //初始化，为寻找订阅方法做准备
        void initForSubscriber(Class<?> subscriberClass) {
            this.subscriberClass = clazz = subscriberClass;
            skipSuperClasses = false;
            subscriberInfo = null;
        }

        //回收，清空集合和恢复初始状态
        void recycle() {
            subscriberMethods.clear();
            anyMethodByEventType.clear();
            subscriberClassByMethodKey.clear();
            methodKeyBuilder.setLength(0);
            subscriberClass = null;
            clazz = null;
            skipSuperClasses = false;
            subscriberInfo = null;
        }
}
```
这里面skipSuperClasses默认为false，说明默认是不忽略父类的，也就说明对象A注册到了事件中心，也将其向上的继承结构注册到了事件中心，不过想想也好理解，因为EventBus要求订阅方法必须是public、non-staic、non-abstract，那么父类的这些方法也就是子类的，因此需要向上查找。关于订阅方法的签名要求，下面会有源码说明。  
查找的核心逻辑是这个循环：  
```
 while (findState.clazz != null) {
            //在单个类中查找订阅方法
            findUsingReflectionInSingleClass(findState);
            //将当前类置为其父类
            findState.moveToSuperclass();
        }
```
### SubscriberMethodFinder.findUsingReflectionInSingleClass()  
```
private void findUsingReflectionInSingleClass(FindState findState) {
        Method[] methods;
        //Step 1:获取该类声明的方法
        try {
            // This is faster than getMethods, especially when subscribers are fat classes like Activities
            methods = findState.clazz.getDeclaredMethods();
        } catch (Throwable th) {
            // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
            methods = findState.clazz.getMethods();
            findState.skipSuperClasses = true;
        }
        //Step 2：遍历方法
        for (Method method : methods) {
            //获取方法的修饰符
            int modifiers = method.getModifiers();
            //case：修饰符合格
            if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                //获取方法参数类型列表
                Class<?>[] parameterTypes = method.getParameterTypes();
                //case：参数长度合格
                if (parameterTypes.length == 1) {
                    Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                    //case:方法被@Subscribe注解修饰了
                    if (subscribeAnnotation != null) {
                        //获取参数类型，即事件类型
                        Class<?> eventType = parameterTypes[0];
                        
                        //case:可以添加            
                        if (findState.checkAdd(method, eventType)) {
                            ThreadMode threadMode = subscribeAnnotation.threadMode();
             //将Method和注解的信息封装到SubscriberMethod保存到FindState中
                            findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                    subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
                    }
                }
                //case :参数长度不合格
                else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                    String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                    throw new EventBusException("@Subscribe method " + methodName +
                            "must have exactly 1 parameter but has " + parameterTypes.length);
                }
            }
            //case: 修饰符不合格
            else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException(methodName +
                        " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
            }
        }
    }
```
首先，从抛出的异常，可以看到几点：  
1. @Subscribe修饰的方法只能有一个参数  
2. @Subscriber修饰的方法必须是public、non-static、non-abstract  
  当符合了条件并且是@Subscribe注解修饰的方法，如果checkAnd()返回true，那么将Method和注解信息封装成SubscriberMethod保存到FindState中的列表中。  
#### FindState.checkAnd()  
```
boolean checkAdd(Method method, Class<?> eventType) {
            // 2 level check: 1st level with event type only (fast), 2nd level with complete signature when required.
            // Usually a subscriber doesn't have methods listening to the same event type.
            Object existing = anyMethodByEventType.put(eventType, method);
            //这个订阅方法是该事件类型的第一个，直接返回true
            if (existing == null) {
                return true;
            }
            //两个以上的方法订阅了同一事件类型
            else {
                if (existing instanceof Method) {
                    //case：检查不过关，抛出异常
                    if (!checkAddWithMethodSignature((Method) existing, eventType)) {
                        // Paranoia check
                        throw new IllegalStateException();
                    }
                    //以FinsState占位Map中Method对象
                    // Put any non-Method object to "consume" the existing Method
                    anyMethodByEventType.put(eventType, this);
                }
                //如果上一次existing是FinsState，那么再检查一次
                return checkAddWithMethodSignature(method, eventType);
            }
        }
```
从注解中可以看到，执行两个层次的检查：  
1. 第一层检查，只检查事件类型；如果该事件类型在该类中第一次出现，那么直接返回true；
2. 第二层检查，需要完整的检查方法签名，这种情况发生在该类中有多个方法同时订阅了某一事件类型。  

这里我们需要分析，当一个事件类型出现了两个及其以上的订阅方法时，就会进入到二层检查;而从代码中可以看到，如果有多个订阅同一事件的方法，那么existing将会在method和findstate中来回切换。

#### FindState.checkAndWithMethodSignature()  
```
private boolean checkAddWithMethodSignature(Method method, Class<?> eventType) {
            methodKeyBuilder.setLength(0);
            methodKeyBuilder.append(method.getName());
            methodKeyBuilder.append('>').append(eventType.getName());

            String methodKey = methodKeyBuilder.toString();
            Class<?> methodClass = method.getDeclaringClass();
            Class<?> methodClassOld = subscriberClassByMethodKey.put(methodKey, methodClass);
            if (methodClassOld == null || methodClassOld.isAssignableFrom(methodClass)) {
                // Only add if not already found in a sub class
                return true;
            } else {
                // Revert the put, old class is further down the class hierarchy
                subscriberClassByMethodKey.put(methodKey, methodClassOld);
                return false;
            }
        }
```
这里可以看到subscriberClassByMethodKey这个Map中的Key的形式是MethodName>EventType，在同一个类中，MethodName肯定是不相同的，那么key就不会相同，那么methodClassOld将一直为null，那么该方法将一直返回true；如果父类中也有该方法并且也是同一事件的订阅方法，那么在查找父类的订阅方法时，methodClassOld将不为null。  
至此，只要checkAnd返回true，那么将一直向FindState中添加订阅方法，而一旦父类中发现了相同的方法，那么不添加，因此子类中已经添加过了。  
### FindState.moveToSuperClass
在单个类中查找完订阅方法，将调用moveToSupperClass()将clazz字段移到父类，其定义如下：  
```
void moveToSuperclass() {
            //case：跳过父类，clazz直接置为null
            if (skipSuperClasses) {
                clazz = null;
            }
            //case：不跳过父类
            else {
                //指向父类
                clazz = clazz.getSuperclass();
                String clazzName = clazz.getName();
                //跳过系统类
                /** Skip system classes, this just degrades performance. */
                if (clazzName.startsWith("java.") || clazzName.startsWith("javax.") || clazzName.startsWith("android.")) {
                    clazz = null;
                }
            }
        }
```
当把订阅者整个继承结构的订阅方法找完之后，调用了getMethodsAndRelease()方法，该方法的定义如下：  
### SubscriberMethodFinder.getMethodsAndRelease()  
```
 private List<SubscriberMethod> getMethodsAndRelease(FindState findState) {
        List<SubscriberMethod> subscriberMethods = new ArrayList<>(findState.subscriberMethods);
        findState.recycle();
        synchronized (FIND_STATE_POOL) {
            for (int i = 0; i < POOL_SIZE; i++) {
                if (FIND_STATE_POOL[i] == null) {
                    FIND_STATE_POOL[i] = findState;
                    break;
                }
            }
        }
        return subscriberMethods;
    }
```
该方法主要完成两步：  
1. 从FindState中把checkAnd()返回结果为true时保存的SubScriberMethod取出；
2. 回收FindState。  

至此，获取到了订阅者中的所有订阅方法，下一步是将这些信息保存到事件中心，以备后续查找进行分发。  


### 例子  
下面以一个例子，说明子类重载父类的订阅方法时，父类中的方法将不再作用。  
```
public class Subscriber {

    @Subscribe
    public void read(Magazine magazine){

        Log.i(MagazineSubscriber.TAG,"superreadMagazine(): "+magazine.toString());

    }
}

ublic class MagazineSubscriber extends Subscriber{

    public static final String TAG="MagezineSubscriber";

    public MagazineSubscriber() {
        super();
        EventBus.getDefault().register(this);
    }

    @Override
    @Subscribe
    public void read(Magazine magazine) {
        Log.i(TAG,"read: "+magazine.toString());

    }

    @Subscribe
    public void readMagazine1(Magazine magazine){

        Log.i(TAG,"readMagazine1(): "+magazine.toString());

    }

    @Subscribe
    public void readMagazine2(Magazine magazine){

        Log.i(TAG,"readMagazine2(): "+magazine.toString());

    }

}
```
在Activity A中初始化MagazineSubscriber，Activity B中发布一个Magezine事件，Log日志如下：  
```
 read: Magazine{title='hello', content='world'}
    readMagazine1(): Magazine{title='hello', content='world'}
    readMagazine2(): Magazine{title='hello', content='world'}
```
可以看到父类中的方法没有打出日志。  
## EventBus.subscribe()  
在找到了订阅方法后，需要将其保存起来，subscribe()方法是在synchronized同步块中的。源码如下：  
```
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        //获取事件类型
        Class<?> eventType = subscriberMethod.eventType;
        //将订阅者与订阅方法组装成元祖
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        //获取事件类型的所有Subscription
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        //case:第一次访问，创建一个新的list
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            //该订阅元组已经出现过，抛出异常
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        //按照优先级排序
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }
        
        //获取订阅者里所有的订阅事件类型
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        //case:如果该订阅方法是Sticky的
        if (subscriberMethod.sticky) {
             //eventInheritance默认是false
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
```
上面的代码主要做两步：  
1. 根据事件类型获取订阅元组列表，按照优先级顺序保存到List中；  
2. 处理事件是Sticky的情况;可以看到最终都调用checkPostStickyEventToSubscription()方法；关于Sticky事件在[EventBus配置、粘性事件、优先级和取消事件分发](https://blog.csdn.net/qq_19431333/article/details/81173432)博客中有介绍。  
  一般来说，如果是Sticky事件，那么stickyEvents将会有结果的，下面看一下checkPostStickyEvevtToSubscription。  
### EventBus.checkPostEventToSubscription()  
```
private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
        if (stickyEvent != null) {
            // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
            // --> Strange corner case, which we don't take care of here.
            postToSubscription(newSubscription, stickyEvent, isMainThread());
        }
    }
    
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
可以看到将会调用postToSubscription()方法，这里有涉及线程分发的选项，关于线程分发的知识，可以参考[EventBus的线程分发](https://blog.csdn.net/qq_19431333/article/details/80992088)，下一篇准备介绍线程分发的源码。这里就以POSTING case往下继续看。  
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
可以看到，这儿就是通过反射去调用订阅者的订阅方法。  
至此，可以分析完了订阅者是如何将自己订阅到事件中心的，要点有如下几点：  
1. EventBus保存了订阅者以及其父类中所有@Subscribe注解了的方法；
2. 订阅者+订阅方法是一个元组；
3. 如果事件是Sticky的，那么将使用反射进行调用；如果不是Sticky的，那么保存在EventBus中的List中。  

> 由此带来的思考，由于EventBus保存了所有的订阅信息：订阅者+订阅方法，会不会占用内存很大？  

# 发布者发布事件，事件如何到订阅方法的  
其实看完上面的代码，应该有个大体思路了，东西都保存在了EventBus中，发布者发完事件，EventBus根据事件去找到所有订阅方法，然后反射调用就OK了，下面我们将实践看一下，是不是这么一个步骤。  
## EventBus.post()  
一切从发布者的post()方法说起，源码如下：  
```
private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
        @Override
        protected PostingThreadState initialValue() {
            return new PostingThreadState();
        }
    };
    
    final static class PostingThreadState {
        final List<Object> eventQueue = new ArrayList<>();
        boolean isPosting;
        boolean isMainThread;
        Subscription subscription;
        Object event;
        boolean canceled;
    }

    public void post(Object event) {
        //每个线程创建一个PostingThreadState，将事件加入队列
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);
        
        
        if (!postingState.isPosting) {
            //是否在主线程发步的
            postingState.isMainThread = isMainThread();
            //置标志位
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                //将队列中所有事件逐个发布出去
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
```
上面涉及ThreadLocal，不了解的朋友可以参考[ThreadLocal源码分析](https://blog.csdn.net/qq_19431333/article/details/60570103)。PostThreadState会为每个线程创建一份，然后共用。同一个线程中发布的事件都会存到它的List中。 postSingleEvent()就是发送单个事件的方法，

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        //case:事件继承，默认false，需要找出该事件的继承结构
        if (eventInheritance) {
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        //如果没有找到订阅方法消费
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
```
从上面可以看到，核心代码是调用了postSingleEventForEventType()方法，该方法的代码如下：  
```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        //加锁，因为注册线程会向该Map中插入数据
        synchronized (this) {
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        //如果有该事件类型的订阅元组
        if (subscriptions != null && !subscriptions.isEmpty()) {
            //遍历元组，由于存放的时候是按优先级的，所以这里也按优先级进行消费
            for (Subscription subscription : subscriptions) {
                //置状态
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted = false;
                try {
                    //发布到单个订阅元组
                    postToSubscription(subscription, event, postingState.isMainThread);
                    //是否被中途取消了
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                //如果中断了，那么退出
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }
```
可以看到最终调用了postToSubscription()方法将事件发布给订阅者，该方法在上面已经碰到了，最终是通过反射进行调用的；这样依次将每一个事件完成了发布。  
# 取消事件分发  
上面涉及到一个参数，PostingThreadState的canceled参数，该参数会在取消事件时标志。取消事件分发是在事件消费方法中调用cancelEventDelivery()方法，该方法的限制场景是ThreadMode是POSTING的情景下，下面会说明原因。该方法的代码如下：  
```
public void cancelEventDelivery(Object event) {
        //获取当前线程的PostingThreadState
        PostingThreadState postingState = currentPostingThreadState.get();
        //情况判断，当前不在分发事件，抛出异常
        if (!postingState.isPosting) {
            throw new EventBusException(
                    "This method may only be called from inside event handling methods on the posting thread");
        } else if (event == null) {
            throw new EventBusException("Event may not be null");
        } else if (postingState.event != event) {
            throw new EventBusException("Only the currently handled event may be aborted");
        } else if (postingState.subscription.subscriberMethod.threadMode != ThreadMode.POSTING) {
            throw new EventBusException(" event handlers may only abort the incoming event");
        }
        
        //设置标志位，后续事件将在分发中终止
        postingState.canceled = true;
    }
```
上面说过，每个线程有一个PostingThreadState，而分发方法和订阅方法都会获取PostingThreadState进行标志位的设置来达到消息通信的，这要求必须得是同一个对象，也就是说发布和订阅是在同一个线程中，而ThreadMode为POSTING的情况下，发布和事件消费是在同一个线程中，这儿也能想象该类为啥叫PostingThreadState了。  

# 总结  
经过上面的源码分析，可以理解事件中心是如何保存订阅者的，订阅者为啥只需调用register()方法，其他就可以什么都不管了，因此事件中心会利用反射找出@Subscribe注解了的方法，然后保存起来；发布者为啥只要post()出事件，剩下的就不要管了，因为事件中心会去寻找出之前保存的订阅者以及订阅方法，然后通过反射进行调用。  