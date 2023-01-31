
### 为什么使用线程池
池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。

**使用线程池的好处**

-   **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
-   **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
-   **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 如何创建线程池

《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

Executors 返回线程池对象的弊端如下：

-   **FixedThreadPool 和 SingleThreadPool** ： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致 OOM。
-   **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM。

#### 方式一：通过构造方法实现
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/ThreadPoolExecutor%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png)

#### 方式二：通过 Executor 框架的工具类 Executors 来实现

我们可以创建三种类型的 ThreadPoolExecutor：

对应 Executors 工具类中的方法如图所示：

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/Executor%E6%A1%86%E6%9E%B6%E7%9A%84%E5%B7%A5%E5%85%B7%E7%B1%BB.png)

1.  newCachedThreadPool()：创建一个可缓存的线程池，调用 execute 将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新线程并添加到线程池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。CachedThreadPool适用于并发执行**大量短期耗时短的任务，或者负载较轻的服务器**；
    
2.  newFiexedThreadPool(int nThreads)：创建固定数目线程的线程池，线程数小于nThreads时，提交新的任务会创建新的线程，当线程数等于nThreads时，提交新的任务后任务会被加入到阻塞队列，正在执行的线程执行完毕后从队列中取任务执行，FiexedThreadPool适用于**负载略重但任务不是特别多**的场景，为了合理利用资源，需要限制线程数量；
    
3.  newSingleThreadExecutor() 创建一个单线程化的 Executor，SingleThreadExecutor**适用于串行执行任务**的场景，每个任务按顺序执行，不需要并发执行；
    
4.  newScheduledThreadPool(int corePoolSize) 创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代 Timer 类。ScheduledThreadPool中，返回了一个ScheduledThreadPoolExecutor实例，而ScheduledThreadPoolExecutor实际上继承了ThreadPoolExecutor。从代码中可以看出，ScheduledThreadPool基于ThreadPoolExecutor，corePoolSize大小为传入的corePoolSize，maximumPoolSize大小为Integer.MAX_VALUE，超时时间为0，workQueue为DelayedWorkQueue。实际上ScheduledThreadPool是一个调度池，其实现了schedule、scheduleAtFixedRate、scheduleWithFixedDelay三个方法，可以实现延迟执行、周期执行等操作；
    
5.  newSingleThreadScheduledExecutor() 创建一个corePoolSize为1的ScheduledThreadPoolExecutor；
    
6.  newWorkStealingPool(int parallelism)返回一个ForkJoinPool实例，ForkJoinPool 主要用于实现“分而治之”的算法，适合于计算密集型的任务。

### ThreadPoolExecutor 类分析

`ThreadPoolExecutor` 类中提供的四个构造方法。我们来看最长的那个，其余三个都是在这个构造方法的基础上产生（其他几个构造方法说白点都是给定某些默认参数的构造方法比如默认制定拒绝策略是什么），这里就不贴代码讲了，比较简单。
```java
/**
 * 用给定的初始参数创建一个新的ThreadPoolExecutor。
 */
public ThreadPoolExecutor(int corePoolSize,
                      int maximumPoolSize,
                      long keepAliveTime,
                      TimeUnit unit,
                      BlockingQueue<Runnable> workQueue,
                      ThreadFactory threadFactory,
                      RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
            throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

### ThreadPoolExecutor参数说明

- **`CorePoolSize`** : 核心线程数定义了最小可以运行的线程数
- **`maximumPoolSize`**: 当队列中存放的任务容量满的时候，当前可以同时运行的线程数量变为最大线程数。当阻塞队列是无界队列, 则`maximumPoolSize`则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入`workQueue`
- **`workQueue`** : 用来保存阻塞的队列，当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。.
-  **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
-   **`unit`** : `keepAliveTime` 参数的时间单位。
-   **`threadFactory`** :executor 创建新线程的时候会用到。
-   **`handler`** :饱和策略。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1671111973043.png)

#### workQueue

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1671715706298.png)

#### 饱和策略

如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，`ThreadPoolTaskExecutor` 定义一些策略:
-   **`ThreadPoolExecutor.AbortPolicy`：** 抛出 `RejectedExecutionException`来拒绝新任务的处理。
-   **`ThreadPoolExecutor.CallerRunsPolicy`：** 由调用线程（提交任务的线程处理），也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
-   **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
-   **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

自定义创建线程池
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolExecutorDemo {

    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;
    private static final Long KEEP_ALIVE_TIME = 1L;
    public static void main(String[] args) {

        //使用阿里巴巴推荐的创建线程池的方式
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_TIME,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 10; i++) {
            //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
            Runnable worker = new MyRunnable("" + i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

## 线程池几种方法对比

### 实现 Runnable 接口和 Callable 接口的区别

`Runnable`自 Java 1.0 以来一直存在，但`Callable`仅在 Java 1.5 中引入,目的就是为了来处理`Runnable`不支持的用例。**`Runnable` 接口** 不会返回结果或抛出检查异常，但是 **`Callable` 接口** 可以。所以，如果任务不需要返回结果或抛出异常推荐使用 **`Runnable` 接口** ，这样代码看起来会更加简洁。

工具类 `Executors` 可以实现将 `Runnable` 对象转换成 `Callable` 对象。（`Executors.callable(Runnable task)` 或 `Executors.callable(Runnable task, Object result)`）。

`Runnable.java`
```java
@FunctionalInterface
public interface Runnable {
   /**
    * 被线程执行，没有返回值也无法抛出异常
    */
    public abstract void run();
}
```

`Callable.java`
```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * 计算结果，或在无法这样做时抛出异常。
     * @return 计算得出的结果
     * @throws 如果无法计算结果，则抛出异常
     */
    V call() throws Exception;
}
```

### 执行 execute()方法和 submit()方法的区别

-   **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
-   **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get(long timeout，TimeUnit unit)`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### `shutdown()`  和  `shutdownNow()` 的区别
-   **`shutdown（）`** :关闭线程池，线程池的状态变为 `SHUTDOWN`。线程池不再接受新任务了，但是队列里的任务得执行完毕。
-   **`shutdownNow（）`** :关闭线程池，线程的状态变为 `STOP`。线程池会终止当前正在运行的任务，并停止处理排队的任务并返回正在等待执行的 List。

###  `isTerminated()` 和 `isShutdown()` 区别
-   **`isShutDown`** 当调用 `shutdown()` 方法后返回为 true。
-   **`isTerminated`** 当调用 `shutdown()` 方法后，并且所有提交的任务完成后返回为 true


## 线程池实践

### 给线程池命名
初始化线程池的时候需要显示命名（设置线程池名称前缀），有利于定位问题。

默认情况下创建的线程名字类似 pool-1-thread-n 这样的，没有业务含义，不利于我们定位问题。

1. **利用guava 创建`ThreadFactory`**
```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue, threadFactory)
```

2. **自定义**

```java
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final ThreadFactory delegate;
    private final String name;
    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(ThreadFactory delegate, String name) {
        this.delegate = delegate;
        this.name = name; // TODO consider uniquifying this
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = delegate.newThread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }
}
```

### 线程池动态监控
- [Java线程池实现原理及其在美团业务中的实践 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [dynamic-tp](https://github.com/dromara/dynamic-tp?spm=a2c6h.12873639.article-detail.7.35382f48TQbhxp)
- [Hippo4j](https://github.com/opengoofy/hippo4j)

### 多线程处理数据
[线程池多线程快速处理List集合](https://blog.csdn.net/qililong88/article/details/114320641)
[ 多线程的利器：CompletableFuture 你还可以这样使用多线程_Java编程Code的博客-CSDN博客](https://blog.csdn.net/qq_39664892/article/details/128297003#:~:text=1%20%E9%81%8D%E5%8E%86list%E9%9B%86%E5%90%88%EF%BC%8C%E6%8F%90%E4%BA%A4CompletableFuture%E4%BB%BB%E5%8A%A1%EF%BC%8C%E6%8A%8A%E7%BB%93%E6%9E%9C%E8%BD%AC%E6%8D%A2%E6%88%90%E6%95%B0%E7%BB%84%202%20%E5%86%8D%E6%8A%8A%E6%95%B0%E7%BB%84%E6%94%BE%E5%88%B0CompletableFuture%E7%9A%84allOf,%28%29%E6%96%B9%E6%B3%95%E9%87%8C%E9%9D%A2%203%20%E6%9C%80%E5%90%8E%E8%B0%83%E7%94%A8join%20%28%29%E6%96%B9%E6%B3%95%E9%98%BB%E5%A1%9E%E7%AD%89%E5%BE%85%E6%89%80%E6%9C%89%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%88%90)

### 合理设置线程池参数
在工程实践中，通常使用下述公式来计算核心线程数：

nThreads=(w+c)/c*n*u=(w/c+1)*n*u  

其中，w为等待时间，c为计算时间，n为CPU核心数（通常可通过 Runtime.getRuntime().availableProcessors()方法获取），u为CPU目标利用率（取值区间为[0, 1]）；在最大化CPU利用率的情况下，当处理的任务为计算密集型任务时，即等待时间w为0，此时核心线程数等于CPU核心数。

  
上述计算公式是理想情况下的建议核心线程数，而不同系统/应用在运行不同的任务时可能会有一定的差异，因此最佳线程数参数还需要根据任务的实际运行情况和压测表现进行微调。

## 线程池可能带来的问题

1.  频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。
2.  对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。
3.  系统无法合理管理内部的资源分布，会降低系统的稳定性。

## 参考文章
- [8000字详解Thread Pool Executor - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/593365020)
- [Java 并发常见面试题总结（下） (javaguide.cn)](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B%E6%B1%A0)
- [Java线程池实现原理及其在美团业务中的实践 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [合理使用线程池以及线程变量 -淘宝技术](https://mp.weixin.qq.com/s/BdVqvm2wLNv05vMTieevMg)