---
title: EventBus 源码分析与踩坑指南
date: 2017-05-22 13:54:49
tags: [Android]
categories:
- Android
---

EventBus是一个使用发布者/订阅者模式 并且低耦合的Android开源库。 EventBus只需几行代码即可实现中央通信解耦类：简化代码，删除依赖关系，加快应用程序开发速度。

<!-- more -->

## EventBus 源码分析

### EventBus 构造器

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

使用的是传统双重检查构造的方法，使得在任意线程中调用构造器都是线程安全的。

默认构造器

```
public EventBus() {
    this(DEFAULT_BUILDER);
}
```

其中传入的默认构造器为：

```
EventBus(EventBusBuilder builder) {
    logger = builder.getLogger();
    //以被订阅事件类为名为 key 记录注册了哪些 onEvent() 方法
    subscriptionsByEventType = new HashMap<>();
    //以注册的类为键值，记录该类所注册的所有事件类型，值为一个Event的class对象的列表
    typesBySubscriber = new HashMap<>();
    //记录了所有粘性事件
    stickyEvents = new ConcurrentHashMap<>();
    //三个Poster, 负责在不同的线程中调用订阅者的方法
    mainThreadSupport = builder.getMainThreadSupport();
    mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
    backgroundPoster = new BackgroundPoster(this);
    asyncPoster = new AsyncPoster(this);
    indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
    //方法的查找类，用于查找某个订阅者类中有哪些注册的方法
    subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
            builder.strictMethodVerification, builder.ignoreGeneratedIndex);
    //一些开关       
    logSubscriberExceptions = builder.logSubscriberExceptions;
    logNoSubscriberMessages = builder.logNoSubscriberMessages;
    sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
    sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
    throwSubscriberException = builder.throwSubscriberException;
    eventInheritance = builder.eventInheritance;
    //线程池
    executorService = builder.executorService;
}
```

对于一个类中会有的多个方法去监听事件，`Subscription`封装了一个注册信息，如下

 ```
final class Subscription {
    //订阅的类
    final Object subscriber;
    //订阅方法封装
    final SubscriberMethod subscriberMethod;
    /**
     * Becomes false as soon as {@link EventBus#unregister(Object)} is called, which is checked by queued event delivery
     * {@link EventBus#invokeSubscriber(PendingPost)} to prevent race conditions.
     */
    volatile boolean active;
}

public class SubscriberMethod {
    //方法
    final Method method;
    //订阅线程
    final ThreadMode threadMode;
    //订阅的事件类的 class
    final Class<?> eventType;
    //事件优先级
    final int priority;
    //是否是粘性事件
    final boolean sticky;
    /** Used for efficient comparison */
    //比较用到的字符串
    String methodString;
}
```

类和方法名唯一确定一条注册信息，Subscription.active 唯一确定该注册信息是否有效。
`subscriptionsByEventType` 以事件的类名为 key ，value 为一个处理事件类型为该键值的 Subscription 的列表。
`typesBySubscriber`根据一个订阅者记录了注册了哪些事件。
这二者从不同维度上记录的订阅事件和被订阅者之间的关系，在使用的注册，调度中更为高效。

### 注册

注册的 `register()` 方法如下：

```
public void register(Object subscriber) {
    //订阅者的类名
    Class<?> subscriberClass = subscriber.getClass();
    //找到所有订阅者订阅的事件，并包装返回
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    //遍历这些事件
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

遍历之后的订阅方法如下：

```
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    Class<?> eventType = subscriberMethod.eventType;
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList<>();
        subscriptionsByEventType.put(eventType, subscriptions);
    } else {
        if (subscriptions.contains(newSubscription)) {
            throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                    + eventType);
        }
    }

    int size = subscriptions.size();
    for (int i = 0; i <= size; i++) {
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }

    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
    if (subscribedEvents == null) {
        subscribedEvents = new ArrayList<>();
        typesBySubscriber.put(subscriber, subscribedEvents);
    }
    subscribedEvents.add(eventType);

    if (subscriberMethod.sticky) {
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

代码很长，逻辑很简单，先构建了`Subscription`了对象，分别放入两个数据结构里。
对于`subscriptionsByEventType`先查再加，如有重复说明一个类中有多个相同的订阅事件，注意事件的优先级，放入列表。如果添加过则抛出异常。
对于`typesBySubscriber`先查再加。
最后检查订阅者注册时是否已存在粘性事件，处理即可。

### 反注册

```
    public synchronized void unregister(Object subscriber) {
        List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
        if (subscribedTypes != null) {
            for (Class<?> eventType : subscribedTypes) {
                unsubscribeByEventType(subscriber, eventType);
            }
            typesBySubscriber.remove(subscriber);
        } else {
            logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
        }
    }
```

我们看到其中最主要的是`unsubscribeByEventType`这个方法，继续查看


```
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}
```

即从 `subscriptionsByEventType`找到所有的订阅事件包装类，分别更新这些包装类。

#### 事件分发

```
public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    eventQueue.add(event);

    if (!postingState.isPosting) {
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
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

如注释所说，`post()`是将事件发送到 EventBus ,具体什么时候调用时由 EventBus 调度，我们继续查看原理。
因为发布事件的线程即调用`post()`的线程与调用订阅者方法的线程不同，所以设计了一个`PostingThreadState`来保存发送的状态。
`PostingThreadState`的结构如下：

```
final static class PostingThreadState {
    //用来保存当前线程需要发送的事件
    final List<Object> eventQueue = new ArrayList<>();
    boolean isPosting;
    boolean isMainThread;
    Subscription subscription;
    Object event;
    boolean canceled;
}
```

故发送事件中最重要的步骤是`postSingleEvent()`:

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
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

`eventInheritance`这个开关是表明是否考虑时间类型的继承关系，默认为true，然后调用`lookupAllEventTypes()`去查找所有订阅了这个时间的Class，最后调用`postSingleEventForEventType()`来判断是否查找到对应的订阅方法，最后将事件通过`postSingleEventForEventType()`方法继续处理：


```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```

首先通过一次 CopyOnWriteArrayList ，每次对List 的修改都会立即同步一份。在这里找到每一个包装类，通过方法`postToSubscription()`来进行分发，

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

这里到了最后的分发事件，在构造方法中构造的四个线程池，加入响应的队列，异步执行，

执行方法为：

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

这里使用了反射，反射调用订阅者的方法，方法存储在subsription的subsribeMethod变量中。


## 优缺点分析
- 优点：洒脱的解藕，在多个界面传递时省去了层层的值传递。
- 缺点：EventBus 直译为事件总线，但是其实总线就是 Bus 也就是公交车。而 RxJava 更像一个滴滴。他直接链接你的两个或多个需要通信的类。传输数据，当然你可以做一个很大的专车，穿梭在所有类之间。
    - 对于 RxJava， Subscriber 明确的知道他 Subscribe 的是谁明确知道要做出什么反应；而对于 EventBus 在项目复杂之后，每个 Observaber 不知道会有什事件下发下来。
    - 不方便调试，对于后续维护来说，如果你没有文档，靠自己找事件被传递到哪里去了还是挺不方便的。


## 踩坑纪要

### `register()` 和 `unregister()` 方法

* 调用时机：

    `register()` 和 `unregister()` 方法的不正确调用是 EventBus 使用中遇到的问题的多数原因，包括且不限于**重复注册**、**注册未收回**、**提前收回**等情况。
    多数情况下，`register()` 方法是伴随着 Activity/Fragment 的生命周期从 onCreate 开始到 onDestroy 结束。如果将 `register()` 和 `unregister()`放在 onStart/onStop 中执行，那就不要怪页面切到后台/不可见时订阅的事件不执行了。有时需要控制不可见的页面不接受事件，则需要放在 onStart/onStop 中执行。
    如果不需要后台的页面立即执行事件，也可以使用粘性广播，当页面重新切回前台时，在 onStart 重新执行额时候会注册订阅者，会立马收到之前发送的粘性广播并执行。
    放在 onCreate 方法中就一定可行么？也不尽然，因为 onCreate 也有可能执行多次，如果一个页面已经存在，多次 startActivity 会创建多个 Activity 实例，就会执行多次，所以调用的时候要注意。
* 收取粘性广播更新 UI 的时候注意页面控件是否初始化完成，先初始化控件，再注册粘性广播执行。

### 对象生成多次

订阅者是对象的实例，当程序中通过种种情况使用的实例不是当时注册 EventBus 的那个实例了（包括且不限于**序列化于反序列化造成另外对象**、**Activity/Fragment 销毁后重建导致前后不一致**、**多级传递把自己传晕了导致对象变了(本人亲历)**等等，遇事不决打一下 Log 看看是否是同一个对象吧。

### 订阅事件的执行线程

订阅事件的时候会判断订阅者的 ThreadMode ，从而决定在什么线程下执行事件响应函数。
* PostThread：默认的 ThreadMode，表示在执行 Post 操作的线程直接调用订阅者的事件响应方法，不论该线程是否为主线程（UI 线程）。当该线程为主线程时，响应方法中不能有耗时操作，否则有卡主线程的风险。适用场景：对于是否在主线程执行无要求，但若 Post 线程为主线程，不能耗时的操作；
* MainThread：在主线程中执行响应方法。如果发布线程就是主线程，则直接调用订阅者的事件响应方法，否则通过主线程的 Handler 发送消息在主线程中处理——调用订阅者的事件响应函数。显然，MainThread类的方法也不能有耗时操作，以避免卡主线程。适用场景：必须在主线程执行的操作；
* BackgroundThread：在后台线程中执行响应方法。如果发布线程不是主线程，则直接调用订阅者的事件响应函数，否则启动唯一的后台线程去处理。由于后台线程是唯一的，当事件超过一个的时候，它们会被放在队列中依次执行，因此该类响应方法虽然没有PostThread类和MainThread类方法对性能敏感，但最好不要有重度耗时的操作或太频繁的轻度耗时操作，以造成其他操作等待。适用场景：操作轻微耗时且不会过于频繁，即一般的耗时操作都可以放在这里；
* Async：不论发布线程是否为主线程，都使用一个空闲线程来处理。和BackgroundThread不同的是，Async类的所有线程是相互独立的，因此不会出现卡线程的问题。适用场景：长耗时操作，例如网络访问。

### 注意 EventBus 事件的混淆

EventBus是采用反射机制调用的绑定的方法，如果混淆则无法找到了。
