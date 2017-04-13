## 多线程——Executors和ThreadPoolExecutor
### 1.Executors简介：
>1.Executors： 
>>- java.util.concurrent.Executors，是一个辅助类，并没有实现或继承Executor接口或者其子接口和实现类。
>>- 不可实例化
>>- 在jdk5开始的加入的

```
/**
* @since 1.5
*/
public class Executors {
    ....
    
    /** Cannot instantiate. */
    private Executors() {}
}
```
>2.Executors类包含许多静态方法，一般使用其创建线程池

### 2.线程创建工厂接口 ThreadFactory和线程池创建类ThreadPoolExecutor：
在介绍Executors之前，先简单介绍下Executors中创建线程是如何创建的的
>1.Executors线程的创建工作是由静态内部类 DefaultThreadFactory来实现的
```
 static class DefaultThreadFactory implements ThreadFactory {
        .......
        //实现 ThreadFactory 接口方法
        public Thread newThread(Runnable r) {
            ....
        }
    }
```
>2.在Executors中创建常见的几种线程池，底层都是ThreadPoolExecutor 类实现的。如下，ThreadPoolExecutor类实例化需要的各个参数，其中ThreadFactory就是创建线程的工厂
```
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
>3.ThreadPoolExecutor对象初始化的参数介绍：
>>- corePoolSize 核心池大小：0
>>- maximumPoolSize 线程数的最大值：Integer.MAX_VALUE
>>- keepAliveTime：当线程池线程的数量大于corePoolSize，在超过keepAliveTime时间后，会将空闲线程删除，直到线程池数量不大于corePoolSize
>>- TimeUnit：keepAliveTime 的时间单位
>>- BlockingQueue：保持任务队列的Runnable任务（一般使用默认）
>>- ThreadFactory：线程创建工厂实现（一般使用默认）
>>- RejectedExecutionHandler：线程池的饱和,出现异常的处理（一般使用默认）
>>>- RejectedExecutionHandler 线程池的饱和策略:
>>>>- AbortPolicy：直接抛出异常，默认策略；
>>>>- CallerRunsPolicy：用调用者所在的线程来执行任务；
>>>>- DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
>>>>- DiscardPolicy：直接丢弃任务；

>4.ThreadPoolExecutor各参数大小关系：
>>1、当任务数 TM 小于 corePoolSize CP 和 maximumPoolSize MP 时，任务被马上创建线程去执行，不放入大小为BQ的 BlockingQueue,其他参数忽略。

>>2、当 TM > CP ，TM < MP 和 BQ 时，超出CP的任务放入BlockingQueue（不是同步队列）中等待执行。

>>3、当 2 中的 BlockingQueue 为同步队列 SynchronousQueue 时，超出 CP 的任务不会被放入队列，而是直接创建线程去执行，完成之后的线程超出keepAliveTime时间将被回收。

>>4.当 TM > CP,TM > MP 和 BQ,maximumPoolSize和keepAliveTime参数失效，超出CP 的任务将被放入BlockingQueue中等待执行。

>>5.当 4 中的 BlockingQueue 为同步队列 SynchronousQueue 时，只处理MP任务，其他任务不在处理，抛出异常。

>>6.当TM > BQ & MP 时，根据丢弃策略进行任务丢弃。业务场景：秒杀系统，

>>7.当 6 中的所有任务执行完成之后，在心跳超时时 MP 会降落，不大于CP数量。业务场景：秒杀系统

>5.在Executors中，常见线程池的创建一般有四种，每一种都有两个创建方法，一般是不需要指定ThreadFactory的，如果有必要，++可自己实现ThreadFactory接口，并在创建线程池时指定，并进行线程异常的处理++
>>- 需要制定ThreadFactory的，
>>- 不指定，即使用Executors类内部默认实现的ThreadFactory，
>>- 下面是一个自定义线程工厂的示例：

```
 /**
     * 自定义线程工厂
     */
    public class MyThreadFactory implements ThreadFactory{
        @Override
        public Thread newThread(Runnable r) {
            Thread newThread = new Thread(r);
            // 设置线程名
            newThread.setName("重命名线程名字：" + Thread.currentThread().getName());
            // 使用内部类进行异常处理
            newThread.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
                @Override
                public void uncaughtException(Thread t, Throwable e) {
                    System.out.println("解决异常！！！" + t.getName() + e.getMessage());
                }
            });
            return newThread;
        }
    }
```



### 3.Executors常见创建线程池方法简介：
由第二点描述的情况来看，Executors创建四类线程池只是参数的调整而已，下面是四类常用线程池的创建方法
>1.创建线程数不限的线程池：newCachedThreadPool();
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
>2.创建线程数固定的线程池：newFixedThreadPool(int nThreads);

```
 public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
>3.创建单例线程池：newSingleThreadExecutor();

```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
>> 1、创建单线程和newSingleThreadExecutor有什么区别？
>>> 1、newSingleThreadExecutor在任务执行完成之后还是会一直存在的，new的单线程执行完就完了，当需要时还是要重新创建，消耗性能

>>> 2、代码容易维护，只需要调用，无需到处创建

>>> 3、能够有序的执行任务

>>> 4、虽然是单线程处理，一旦线程因为处理异常等原因终止的时候，ThreadPoolExecutor会自动创建一个新的线程继续进行工作

>> 2、缺点：
>>> 1、和FixedThreadPool相同，当处理任务无限等待的时候会造成内存问题。

>4.创建定时执行的线程池：newScheduledThreadPool(int corePoolSize);

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

// ScheduledThreadPoolExecutor 继承 ThreadPoolExecutor
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
              new DelayedWorkQueue());
    }
```
### 4.ThreadPoolExecutor常用方法简介：
>1.shutdown()和shutdownNow()方法：
>>1、shutdown方法：执行之后所有线程的状态变成 SHUTDOWN,未执行完任务的线程会继续完成任务，新的任务不会被执行,也不会添加新的任务，不会阻塞，主线程马上结束，当所有任务完成之后线程池销毁

>>2、shutdownNow方法：执行之后所有线程变成stop状态，会尽可能去中断所有线程，但不一定成功，最好是进行中断检查（if(Thread.currentThread().isInterrupted() == true) 或者人为抛出异常）

>2.ThreadPoolExecutor构造方法：
>>> 1、ThreadPoolExecutor方法的BlockingQueue一般有三种队列，ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue。
>>>- 前两种队列可以设置大小，SynchronousQueue不可以
>>>- 设置大小后，如果任务数量超过maximumPoolSize和队列大小之和，超出的任务会被拒绝，报异常。

>>> 2、ThreadPoolExecutor的拒绝策略：
>>>- AbortPolicy：当任务添加被拒绝时，抛出异常RejectedExecutionException。
>>>- CallerRunsPolicy：拒绝时，会调用线程池的Thread线程对象去处理拒绝任务
>>>-DiscardOldestPolicy：拒绝时，替换等待队列中等待时间最久的任务。
>>>-DiscardPolicy：拒绝时，直接丢弃拒绝任务

>3.remove()方法：删除尚未被执行的任务，只能删除execute提交的任务，submit提交的即时是尚未被执行也是不能删除的。

>4.afterExecute()和beforeExecute()：在线程池创建类里面重写这两个方法，可以做一些监控之类的的功能，类似于aop。

>5.注：通过ThreadPoolExecutor的几个属性的set方法设置线程池的一些属性，比如，设置创建后线程池的大小，使用SetCorePoolSize()方法。
