

# JUC

JUC指的是 `java.util.concurrent`  这个包中的内容。

包括原子类，锁，以及一些并发工具类，比如支持并发的map，并发的队列，线程池等等。



## locks

我们从锁开始说起。锁，锁的是什么？共享资源。我们用锁来控制多个线程访问共享资源，可以防止多个线程同时操作共享资源而导致数据不一致的问题。在JDK5之前，我们是通过synchronized来实现锁功能的，但是这个锁不够灵活。在JDK5之后，并发包中新增了Lock接口，提供了可控制的获取释放锁、可中断、可设置超时获取等锁功能。

我们先从使用开始，之后慢慢剖析它的实现。Lock锁的使用很简单：

``` java
Lock lock = new ReentrantLock(); // 定义锁
lock.lock(); // 上锁
try{
    
}finally{
    lock.unlock(); // 解锁
}
```

Lock是一个接口，它定义了获取锁和释放锁的基本操作：

| method                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void lock()                                                  | 阻塞，不可中断。获取锁后，从该方法返回。                     |
| void lockInterruptibly() throws InterruptedException         | 可中断的获取锁。                                             |
| boolean tryLock()                                            | 尝试获取锁，会立即返回结果。能够获取则返回true，不能获取也会立即返回false。 |
| boolean tryLock(long time, TimeUnit unit) throws InterruptedException | 可设置获取锁的超时时间。<br />1. 被中断返回，抛出异常；<br />2. 获得锁返回，true；<br />3. 超时返回，false。 |
| void unlock()                                                | 解锁。                                                       |
| Condition newCondition()                                     | 获取等待通知的组件，与当前锁绑定。只有获得了锁，才能调用该组件的wait()方法释放锁。 |



## 队列同步器



AbstractQueuedSynchronizer，队列同步器是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的先进先出队列来完成资源获取的线程的排队工作。

子类通过继承同步器并实现它的抽象方法来管理同步状态。管理同步状态，无非就是获取状态以及设置状态。

同步器的设计是基于模板方法模式的。AQS对自己的臣民说：“我呢，制定了一些规则，有些你们统一都要做的事情，由我来做；其他的事情，我让你们做你们再做。不管你是独占锁还是共享锁，去找你要锁的线程都要用我的方法去排队、离队。至于排队前要不要偷偷插个队？排队的要排多久，无限制排队，还是限定时间？这个就归你们的管理了。”



获取到锁的线程一定要给他一个勋章，要用勋章来让别人知道获取锁的是他。也就是使用 state来证明，只要state>0，就说明这个锁被别人占了，其他线程排队去。如何让其他线程看到这个state值变了呢？使用volatile修饰。

还要在锁这里给他保留一个爵位，要让锁自己知道获取锁的线程是谁。也就是锁内部使用 exclusiveOwnerThread 来保存当前获取锁的线程。





## 并发容器

**同步容器：**

Java集合主要为4类：List，Map，Set，Queue，线程不安全的：ArrayList，HashMap

JDK早期线程安全的集合Vector，Stack，HashTable

JDK1.2中，还为Collections增加内部Synchronized类创建出线程安全的集合，实现原理synchronized
Collections.synchronizedCollenction/List/Map/Set...

**并发容器**

针对多线程并发访问来进行设计的集合，称为并发容器。

- JDK1.5 之前，JDK提供了线程安全的集合都是同步容器，线程安全，只能串行执行，性能很差。
- JDK 1.5之后，JUC并发包提供了很多并发容器，优化性能，替代同步容器。

**常见并发容器特点**

| List容器                                         |                                                              |
| ------------------------------------------------ | ------------------------------------------------------------ |
| Vector                                           | synchronized实现的同步容器，性能差，适合于**对数据有强一致性**要求的场景 |
| CopyOnWriteArrayList                             | 底层数组实现，使用**复制副本**进行有锁写操作（数据不一致问题），适合读多写少，允许短暂的数据不一致的场景 |
| **Map容器**                                      |                                                              |
| Hashtable                                        | synchronized实现的同步容器，性能差，适合于**对数据有强一致性**要求的场景 |
| <span style='color:red'>ConcurrentHashMap</span> | 底层数组+链表+红黑树(JDK1.8)实现，对table数组entry加锁（synchronized）。存在一致性问题。适合存储**数据量少，读多写少**，允许短暂的数据不一致的场景。 |
| ConcurrentSkipListMap                            | 底层跳表实现，使用CAS实现无锁读写操作。适合与存储**数据量大，读写频繁**，允许短暂的数据不一致的场景。 |
| **Set容器**                                      |                                                              |
| CopyOnWriteArraySet                              | 底层数据实现的无序Set                                        |
| ConcurrentSkipListSet                            | 底层基于调表实现的有序Set                                    |



### ConcurrentHashMap







### CopyOnWriteArrayList





### 队列

通过队列可以很容易实现数据共享，并且解决上下游处理速度不匹配的问题，典型的生产者消费者模式。

队列中的读写等线程安全问题由队列负责处理。







## 线程池



### 核心参数&原理

线程池的核心参数

| 参数名        | 类型                     | 含义                                                         |
| ------------- | ------------------------ | ------------------------------------------------------------ |
| corePoolSize  | int                      | 核心线程数，空闲也不会回收                                   |
| maxPoolSize   | int                      | 最大线程数                                                   |
| keepAliveTime | long                     | 保持存活时间，针对于非核心线程                               |
| workQueue     | BlockingQueue            | 任务存储队列，直接交换队列SynchronousQueue，无界队列LinkedBlockingQueue，有界队列ArrayBlockingQueue |
| threadFactory | ThreadFactory            | 线程池创建新线程的工厂类                                     |
| Handler       | RejectedExecutionhandler | 线程无法接受任务时的拒绝策略                                 |



### 自动创建线程

- newFixedThreadPool：固定数量线程池，无界任务阻塞队列
- newSingleThreadExecutory：一个线程的线程池，无界任务阻塞队列
- newCacheThreadPool：可缓存线程的无界线程池，可以自动回收多余线程
- newScheduledThreadPool：定时任务线程池



### 手动创建线程

有些企业开发规范中会禁止使用快捷方式创建线程池，要求使用标准构造器ThreadPoolExecutor创建。

**如何设置线程池大小？**

- CPU密集型：线程数量不能太多，可以设置为CPU核数
- IO密集型：IO密集型CPU使用率不高，可以设置的线程数量多一些，可以设置为CPU核心数的2倍

**拒绝策略：**

拒绝时机：①最大线程和工作队列有限且已经饱和，②Executor关闭时

抛异常策略：AbortPolicy，说明任务没有提交成功

不做处理策略：DiscardPolicy，默默丢弃任务，不做处理

丢弃老任务策略：DiscardOldestPolicy，将队列中存在最久的任务给丢弃

自产自销策略：CallerRunsPolicy，哪个线程提交任务就由哪个线程负责运行

```java
// 使用标准构造器，构造一个普通的线程池
public ThreadPoolExecutor(
        int corePoolSize, // 核心线程数，即使线程空闲（Idle），也不会回收
        int maximumPoolSize, // 线程数的上限
        long keepAliveTime, TimeUnit unit, // 线程最大空闲（Idle）时长
        BlockingQueue workQueue, // 任务的排队队列
        ThreadFactory threadFactory, // 新线程的产生方式
        RejectedExecutionHandler handler) // 拒绝策略
```





## ThreadLocal

### 线程封闭

> **线程封闭**
>
> 当访问共享的可变数据，通常需要使用同步。一种避免使用同步的方式就是不共享数据。如果仅在单线程内访问数据，就不需要同步，这种技术被成为**线程封闭**（Thread Confinement）。
>
> 线程封闭技术的一种常见应用是JDBC的Connection对象。JDBC规范并不要求Connection对象必须是线程安全的。大多数的请求（Servlet请求）都是由单个线程采用同步的方式来处理，并且在Connection对象返回之前，连接池不会把它分配给其他线程，因此，这种连接管理模式在处理请求时，隐含地将Connection对象封闭在线程中。
>
> Java提供的维持线程封闭性的机制：局部变量（栈封闭）  和  ThreadLocal类。
>
> - 栈封闭：只能通过局部变量才能访问对象。基本类型（默认封闭，因为任何方法都无法获得对基本类型的引用），引用类型（需要保证不发布引用）
> - ThreadLocal：这个类能使线程中的某个值与保存值的对象关联起来。提供了get，set方法。get总是返回由当前线程在调用set时设置的最新值。

ThreadLocal使用在什么场景呢？

ThreadLocal通常用于防止对可变的单实例变量或全局变量进行共享。（如何理解这句话呢？一个对象，是单例的，但是它其中的一个变量每个线程不能共享，一种使用方法就是，在线程的最开始创建这个变量的实例，然后沿着各个方法一路传下去；另一种方法就是使用ThreadLocal，每次使用的时候去ThreadLocal里面拿）

```java
/*使用ThreadLocal来维持线程封闭性*/
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
    @Override
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};

public static Connection getConnection(){
    return connectionHolder.get();
}

```

当某个线程初次调用ThreadLocal.get()方法时，就会调用initialValue来获取初始值。从概念上看，可以将`ThreadLocal<T>` 视为包含了Map<Thread,T>对象，保存了特定于该线程的值，但ThreadLocal的实现并非如此。这些特定于线程的值保存在Thread对象中，当线程终止后，这些值会作为垃圾回收。

在实现应用程序框架时大量使用了ThreadLocal。当框架代码需要判断当前运行的是哪一个事务时，只需从这个ThreadLocal对象中读取事务上下文，这种机制可以避免在调用每个方法时都需要传递执行上下文信息，然而这也将使用该机制的代码与框架耦合在一起。

ThreadLocal变量类似于全局变量，它能降低代码的可重用性，并在类之间引入隐含的耦合性。在使用时要格外小心。

### get/set原理

**类ThreadLocal解决的是变量在不同线程中的隔离性，也就是不同线程拥有自己的值，不同线程的值是可以通过ThreadLocal类进行保存的。**

| <span style="display:inline-block;width:80px">方法</span> |                                                              |
| :-------------------------------------------------------- | ------------------------------------------------------------ |
| get()与null                                               | 如果从未在Thread中的Map存储ThreadLocal对象对应的值，则get()方法返回null。`ThreadLocal.ThreadLocalMap threadLocals = null;` |
| set()                                                     | 1.  getMap(thread)   →   ThreadLocal.ThreadLocalMap   ThreadLocalMap是ThreadLocal的内部类<br />2.  createMap(thread,value) → new ThreadLocalMap(this,value)<br />3.  将ThreadLocal对象与value封装进Entry |
|                                                           |                                                              |
|                                                           |                                                              |

get()

```java 
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);  // 1. 这里怎么理解？
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
/*
？答：
1. ThreadLocal里面有一个内部类：ThreadLocalMap
2. ThreadLocalMap里面有一个内部类：Entry<ThreadLocal,value>；
3. ThreadLocalMap里面有一个实例变量table，是Entry[]
4. 也就是ThreadLocalMap本质是一个数组，这个数组的每一项是一个Entry，这个Entry包含“key值ThreadLocal对象”和“value值”。一个ThreadLocal对象只对应一个value。所以可以通过key值直接得到value值
*/
```



set()

```java
// 1. ThreadLocal# set()
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); // 2. ThreadLocal# getMap()
    if (map != null)
        map.set(this, value); // ThreadLocalMap中的key就是当前的ThreadLocal对象
    else
        createMap(t, value); // ThreadLocal# createMap()
}
// 2. ThreadLocal# getMap()
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals; // 3. 线程的实例变量，Thread#ThreadLocal.ThreadLocalMap threadLocals = null;
}
// 3. ThreadLocal# createMap(t, value)
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
// 4. ThreadLocalMap# ThreadLocalMap() 构造方法
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
    }
    
    private Entry[] table;
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY]; // new Entry[16]
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);  // 核心代码
        size = 1;
        setThreshold(INITIAL_CAPACITY);  // threshold = len * 2 / 3;
    } 
}
```



问题：为什么不直接向Thread类中的ThreadLocalMap对象存取数据呢？threadLocals.set？

ThreadLocal.ThreadLocalMap threadLocals = null; threadLocals 默认是包级访问（没有public修饰），不能从外部直接访问，也没有对应的public的get和set方法，只有用同一个包中的类可以访问threadLocals变量。ThreadLocal和Thread都在java.lang包中。



### 弱引用

ThreadLocalMap中的静态内置类Entry是弱引用类型。

```java 
// WeakReference的构造方法中说：创建一个适用于k的弱引用。也就是说会有一个弱引用指向ThreadLocal<?> k
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

只要垃圾回收器扫描时发现弱引用的对象，则不管内存是否足够，都会回收弱引用的对象。也就是只要执行gc操作，ThreadLocal对象就立即销毁，代表key的值ThreadLocal对象会随着gc操作而销毁，释放内存空间，但是value值不会随着gc而销毁，这就会出现内存溢出。

当ThreadLocalMap中的数据不再使用时，要手动执行ThreadLocal.remove()方法，清除数据，释放内存空间，否则会出现内存溢出。





如果是强引用Entry该怎么定义呢？

1. 也就是说Entry是一个单值的value，并不是之前Map里面的Entry，有key有value。

```java 
static class Entry {
    ThreadLocal<?> key;
    Object value;
    Entry(ThreadLocal<?> k,Object v) {
        key = k;  // 这样Entry里面的key就是我们最初直接new的ThreadLocal对象了。
        value = v;
    }
}
```

2. 甚至Entry直接是一个Object就可以了，没必要再包一层。

```java 
static class ThreadLocalMap { 
    // private Entry[] table;
    private Object[] table;
    
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Object[INITIAL_CAPACITY]; // new Entry[16]
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        Object[i] = firstValue;  // 核心代码
        size = 1;
        setThreshold(INITIAL_CAPACITY);  // threshold = len * 2 / 3;
    } 
}
```



我认为，ThreadLocal不一定是一个方法创建的。在一个方法中创建ThreadLocal对象有意义吗？ThreadLocal对象作为类A一个静态变量或者实例变量出现。

这个线程执行完了。但是这个类A的对象还存在。也就是它的ThreadLocal变量还存在。

对象A 和 ThreadLocal变量。

一个线程里面有一个ThreadLocalMap。这个ThreadLocalMap对象，里面维护了一个Entry数组。Entry的每一个对象，弱引用于这个ThreadLocal变量，强引用于value。





## Future

