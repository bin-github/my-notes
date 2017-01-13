## CompletionService接口及实现类简介：
### 1、CompletionService接口：
>1.其唯一的实现类为ExecutorCompletionService,

>2.解决Future的get方法阻塞等待执行结果的缺点，

### 2、ExecutorCompletionService：
>1.ExecutorCompletionService类take方法，
>>- take()方法异步获取多个任务的结果，如果一个任务都没有出结果，take方法也会阻塞等待
>>- take方法实现：线程池将处理后的结果保存在初始化时的阻塞队列中，take方法就依赖于各个阻塞队列中的take方法，当BlockingQueue不为空时，take方法就会获取其中的一个值，为空时阻塞，这就是为什么ExecutorCompletionService中的take方法为什么可以异步获取结果值的原因。
>>- take只能抛出线程字段异常InterruptedException，await方法有可能出现此异常，call方法里面的异常take是无法捕获的，只有get才能捕获
>>- 下面是ArrayBlockingQueue类的take实现源码：

```
// take抛出异常InterruptedException，是因为await方法有可能出现此异常，
 public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return extract();
        } finally {
            lock.unlock();
        }
    }
// 获取BlockingQueue中的一个对象
 private E extract() {
        final Object[] items = this.items;
        E x = this.<E>cast(items[takeIndex]);
        items[takeIndex] = null;
        takeIndex = inc(takeIndex);
        --count;
        notFull.signal();
        return x;
    }
```
>2.poll方法：和take类似，无阻塞，无结果时返回null，
>>- poll()：无阻塞
>>- poll(long timeout, TimeUnit unit)：阻塞等待timeout时间，源码如下：

```
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                if (nanos <= 0)
                    return null;
                // 等待规定的时间
                nanos = notEmpty.awaitNanos(nanos);
            }
            return extract();
        } finally {
            lock.unlock();
        }
    }
```
>3.ExecutorCompletionService使用示例

```
 private static void futureTest() {
        ExecutorService executors = Executors.newCachedThreadPool();
        // 因为setThreadFactory方法是在ExecutorService子类
        // ThreadPoolExecutor中定义的，所以需要强制类型转换
        ThreadPoolExecutor poolExecutor = (ThreadPoolExecutor) executors;
        // 此处重新设置线程工厂
        poolExecutor.setThreadFactory(new ThreadFactory(){
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread();
                thread = (Thread)r;
                thread.setName("重置线程名：" + Thread.currentThread().getName());
                return thread;
            }
        });

        CompletionService cs = new ExecutorCompletionService(poolExecutor);
        Future future = cs.submit(new Callable() {
            @Override
            public Object call() throws Exception {
                System.out.println("我被调用了！！");
                return "lalala";
            }
        });
        // 需要捕获异常
        try {
            String result = (String)future.get();
            String ss = (String)cs.take().get();
            System.out.println(result);
            System.out.println(ss);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

>4.