Posted on 2019-03-07
> 线程池是多线程编程中的核心概念，简单来说就是一组可以执行任务的空闲线程。
>
> 线程是在一个进程中可以执行一系列指令的执行环境，或称运行程序。多线程编程指的是用多个线程并行执行多个任务，虽然程序性能提高，但多线程编程也有缺点 —— 增加了代码复杂度、同步问题、非预期结果和增加创建线程的开销。

### Java 线程池

频繁创建并开启线程开销很大，每次重复这些步骤占用了很大的性能开销，线程在我们使用前一直保存在线程池中，在执行完任务之后，线程会返回线程池等待下次使用。

java.util.concurrent 并发包下有以下接口：

- `Executor` —— 执行任务的简单接口
- `ExecutorService` —— 一个较复杂的接口，包含额外方法来管理任务和 executor 本身
- `ScheduledExecutorService` —— 扩展自 `ExecutorService`，增加了执行任务的调度方法

除了这些接口，还有 `Executors` 类来直接获取实现了上述接口的实例，一般来说，Java 线程池应该包含以下部分：

- 存放工作线程的池子，负责`管理`线程
- 线程工厂，负责`创建`新线程
- 等待执行的任务队列

### `Executors` 类和 `Executor` 接口

`Executors` 类包含许多创建不同类型线程池的方法，`Executor` 是一个简单的线程池接口，只有一个 `execute()` 方法。

他们之间关系如下，创建一个单线程的线程池（lambda 表达式会自动推断成 Runnable 类型）：

```java
Executor executor = Executors.newSingleThreadExecutor();
executor.execute(() -> System.err.println("This is a SingleThreadExecutor test"));
```

#### `Executor` 类继承关系图如下：

<img src="https://tryme.wang/usr/images/sina/5cd95d6d4611d.jpg" alt="image" align="center">

如果有工作线程可用，`execute()` 方法会执行，否则将 Runnable 任务放入队列，等待线程可用。

#### `Executors` 类里的工厂方法可以创建很多类型的线程池：

<img src="https://tryme.wang/usr/images/sina/5cd95d6f2e412.jpg" alt="image" align="center">

- `newSingleThreadExecutor()`：包含单个线程和无界队列的线程池，同一时间只能执行一个任务

  源码：

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  ```

  可以看到 `corePoolSize` 和 `maximumPoolSize` 均为 1（`ThreadPoolExecutor` 详解见下面章节）

- `newFixedThreadPool()`：包含固定数量线程并共享无界队列的线程池，当所有线程处于工作状态，有新任务提交时，任务在队列中等待，直到一个线程变为可用状态

  源码：

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
  ```

  可见，线程数为 `nThreads` 参数

- `newCachedThreadPool()`：只有需要时创建新线程的线程池

  源码：

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  ```

  可以看到 `corePoolSize` 为 0， `maximumPoolSize` 为 Integer 最大值

- `newWorkStealingThreadPool()`：基于工作窃取（work-stealing）算法的线程池。

#### `ExecutorService` 接口

该接口继承自 `Executor`，并加入了几个方法，`Executor` 接口只有一个 `execute()` 方法，看下 `ExecutorService` 的结构图：

<img src="https://tryme.wang/usr/images/sina/5cd95d6fd676c.jpg" alt="image" align="center">

新加了许多 API，`submit()` 方法与 `execute()` 相似，这个方法可以返回一个 `Future` 接口，该接口可以返回 Callable 类型任务的结果及显示任务的执行状态：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Callable<String> task = () -> "done";
Future<String> future = executorService.submit(task);
if (future.isDone()) {
    try {
        System.err.println(future.get());
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

并且当没有任务等待执行是，`ExecutorService` 并不会自动关闭，需要调用 `shutdown()/shutdownNow()` 来显示关闭。

#### ScheduledExecutorService 接口

在 ExecutorService 接口基础上增加了任务调度的方法。

<img src="https://tryme.wang/usr/images/sina/5cd95d713ddf0.jpg" alt="image" align="center">

```java
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
// schedule() 两个方法的参数指定执行的方法、延时和 TimeUnit
scheduledExecutorService.schedule(() -> System.err.println("Delay task"), 2, TimeUnit.SECONDS);
Callable<String> callable = () -> "callable task";
ScheduledFuture<String> future = scheduledExecutorService.schedule(callable, 3, TimeUnit.SECONDS);

// 延时 2 毫秒执行任务，然后每 2 秒重复一次。
scheduledExecutorService.scheduleAtFixedRate(() -> System.err.println("Fixed Rate"), 2, 2000,TimeUnit.MILLISECONDS);
// 延时 2 毫秒后执行第一次，然后在上一次执行完成 2 秒后再次重复执行。
scheduledExecutorService.scheduleWithFixedDelay(() -> System.err.println("Fixed delay"), 2,2000, TimeUnit.MILLISECONDS);
```

#### `ExecutorService` 的实现类 `ThreadPoolExecutor` 

该类所需参数最多的一个构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler){...}
```

其中各个参数的含义：

根据 corePoolSize 和 maximumPoolSize 设置的边界自动调整池大小

- `corePoolSize`  `maximumPoolSize`：当一个新的任务提交，如果线程数小于 corePoolSize，即使其他线程空闲，也会创建一个线程去执行请求；如果线程数大于 corePoolSize，但小于 maximumPoolSize，则只有在线程队列满的情况下才会创建新的线程；如果两个值相等，相当于创建了一个固定大小的线程池；如果设置 maximumPoolSize 为一个无边界的值，例如 Integer.MAX_VALUE，则允许池容纳任意数量的并发任务

- `keepAliveTime` `unit`：如果线程池当前具有多于 corePoolSize 个线程，则如果多余线程空闲时间超过keepAliveTime 会被终止；unit 为时间单位

- `workQueue`：任何 BlockingQueue 都可以传输或保存提交的任务并和线程池大小相互作用，类似于上面那样：

  - 如果少于 corePoolSize 线程正在运行，Executor 总是喜欢添加一个新线程，而不是排队
  - 如果 corePoolSize 或更多的线程正在运行，Executor 总是喜欢排队请求而不是添加一个新的线程
  - 如果请求无法排队，则会创建一个新线程，除非这将超出 maximumPoolSize，否则任务将被拒绝

  排队有三种策略：
  1. 直接切换 一个工作队列的一个很好的默认选择是一个SynchronousQueue ，将任务交给线程，无需另外控制。无线程可用会创建新的，通常需要无限制的 maximumPoolSize

  1. 无界队列 使用无界队列（例如 LinkedBlockingQueue 没有预定容量）会导致当所有 corePoolSize 线程都很忙时，新的任务在队列中等待。 因此，不会再创建超过 corePoolSize 线程。 （因此，最大值大小的值没有任何影响。）每个任务完全独立于其他任务时，因此任务不会影响其他执行; 例如，在网页服务器中。 虽然这种排队风格可以有助于平滑瞬态突发的请求，但是当命令继续达到的平均速度比可以处理的速度更快时，它承认无界工作队列增长的可能性。

  1. 有界队列 有限队列（例如， ArrayBlockingQueue ）有助于在使用有限maxPoolSizes时防止资源耗尽，但可能更难调整和控制。队列大小和线程池大小彼此作用：大队列小线程池占用资源低，导致人为的低吞吐量；使用小型队列通常需要较大的池大小，这样可以使CPU繁忙，但可能会遇到不可接受的调度开销，这也降低了吞吐量。

- `handler`：当执行程序对最大线程和工作队列容量使用有限边界并且饱和时的策略， 提供了四个预定义的处理程序策略：

  1. 在默认 `ThreadPoolExecutor.AbortPolicy` ，处理程序会引发运行RejectedExecutionException后排斥反应

  1. 在 `ThreadPoolExecutor.CallerRunsPolicy` 中，调用execute本身的线程运行任务。 这提供了一个简单的反馈控制机制，将降低新任务提交的速度

  1. 在 `ThreadPoolExecutor.DiscardPolicy` 中 ，简单地删除无法执行的任务

  1. 在 `ThreadPoolExecutor.DiscardOldestPolicy` 中 ，如果执行程序没有关闭，则工作队列头部的任务被删除，然后重试执行（可能会再次失败，导致重复）

- `threadFactory`：创建线程的工厂。

  ```java
  public interface ThreadFactory {
  
      /**
       * Constructs a new {@code Thread}.  Implementations may also initialize
       * priority, name, daemon status, {@code ThreadGroup}, etc.
       *
       * @param r a runnable to be executed by new thread instance
       * @return constructed thread, or {@code null} if the request to
       *         create a thread is rejected
       */
      Thread newThread(Runnable r);
  }
  ```