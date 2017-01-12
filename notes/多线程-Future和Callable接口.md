
### 1.Future简介：
> 1.Future：一个接口，其实现类接受Callable实现类中call方法的返回结果。

>2.Future的不足：
>>- get方法是阻塞的，当所有的提交的线程是结果都完成时才会获得结果
>>- get方法的这种机制，使得任务的完成时有顺序，但是获得结果是确实没法按照完成的顺序来获得的。导致所有结果一起处理，效率低下。
>>-  public V get(long timeout, TimeUnit unit)方法：在规定的最大时间内获取值，如果超时则报异常


### 2.Callable接口简介：
>1.Callable和Runable接口类似：

```
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
public interface Runnable {
    .....
    public abstract void run();
}
```
>2.call方法和run方法相比:
>>- call方法有返回值，并且有异常抛出，在外层可捕获异常
>>- run方法没有返回值，有异常时无法捕获，但可以在线程池中处理异常

### 3.方法小结：
>1.execute（ThreadPoolExecutor类）和submit（ExecutorService接口）方法：
>>- 两者的区别如下：

不同点 |execute|submit
---|---|---
返回值 | 无    | call方法的返回值
异常处理 | 直接抛出，或者在ThreadFactory中自定义处理|可以正常的用try{}catch(){}去处理
可提交的线程类型|runable类型|runable和callable类型
>>- submit提交runable任务的时候，最终将runable任务适配成FutureTask，由execute方法去执行，只是返回值结果为空。
>>- submit的任务是不管是runable还是callable的，最后交由execute方法去执行。

```
// 1 AbstractExecutorService类
public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
// 2 新建一个FutureTask去包装runable   
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
// 3 FutureTask类
public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
    
// 4 最后由ThreadPoolExecutor类的execute方法去执行
public void execute(Runnable command) {}
```
### 4.简单示例：
>1.submit实现了callable的任务;

```
private static void futureTest() {
        ExecutorService executors = Executors.newCachedThreadPool();
        Future future = executors.submit(new Callable() {
            // 此处可抛出异常，在外层捕获处理
            @Override
            public Object call() throws Exception {
                System.out.println("我被调用了！！");
                // 返回值
                return "我是call方法的返回值";
            }
        });
        try {
            String result = (String)future.get();
            System.out.print(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

>2.submit实现了Rrunable的任务;

```
private static void futureTest() {
        ExecutorService executors = Executors.newCachedThreadPool();
        Future future = executors.submit(new Runnable() {
            @Override
            public void run() {
                System.out.println("我被调用了！！");
            }
        });
        try {
            int result = (int)future.get();
            System.out.print(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```


