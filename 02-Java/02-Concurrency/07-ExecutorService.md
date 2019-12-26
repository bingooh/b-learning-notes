## 基本接口
- `Executors` 工具类
- `Executor` 执行任务
- `ExecutorService` 执行任务，返回`Future`
- `ScheduledExecutorService` 执行延迟任务或周期性任务

## ExecutorService
- `ExecutorService`方法
    - `submit()`     新增1个任务
    - `invokeAll()`  新增多个任务，阻塞线程直到全部执行完毕
    - `invokeAny()`  新增多个任务，阻塞线程直到任意1个执行完毕
    - 生命周期方法见线程池

## 线程池
- `ThreadPoolExecutor` -> `ExecutorService`
- `Executors`提供以下方法
    - `newFixedThreadPool()`     固定线程数线程池
    - `newCachedThreadPool()`    无限线程数线程池
    - `newSingleThreadPool()`    单线程数线程池
    - `newScheduledThreadPool()` `schedule`线程池
    - `newWorkStealingPool()`    `work steal`线程池
    - `defaultThreadFactory()`   线程池默认`ThreadFactory`
    - `unconfigurableExecutorService(executor)` 封装为不能修改配置的线程池
- `ThreadPoolExecutor` 配置参数
    - `corePoolSize`     核心线程数,线程池里最少线程数,包括`idle`线程
    - `maximumPoolSize`  最大线程数
    - `keepAliveTime`    超过核心线程数的`idle`线程最大存活时间
    - `workQueue`        存放任务的`BlockingQueue`
    - `threadFactory`    创建线程的工厂类，可自定义线程的`UncaughtExceptionHandler`等
    - `rejectedExecutionHandler` 处理`rejected`任务
    - `allowCoreThreadTimeOut()` 是否允许核心线程超时结束
- `ThreadPoolExecutor`方法
    - `prestartCoreThread()` 启动核心线程，等待执行任务，适用于创建线程池时工作队列已有任务
    - `getQueue()` 获取工作队列，仅用于监控和调试
    - `purge()` 移除已取消的任务
- `ThreadPoolExecutor` 生命周期方法
    - `beforeExecute()` 在每个任务执行前回调
    - `afterExecute()`  在每个任务执行后回调
    - `teminated()`     在线程池结束后回调
    - `shutdown()`      关闭线程池：继续处理已接收的任务，拒绝接收新任务
    - `shutdownNow()`   立刻关闭线程池：尝试停止正在处理的任务，返回所有还未开始处理的任务，拒绝接收新任务
    - `awaitTermination()`    等待线程池结束，线程池处理完已接收的任务后将退出
    - `isTerminated()` 线程池已结束，所有任务已处理
    - `isShutdown()`   线程池已关闭，此时可能还需处理已提交的任务
- `rejectedExecutionHandler` 标准策略
    - `AbortPolicy` 默认策略，直接抛出`RejectedExecutionException`
    - `CallerRunsPolicy` 交给提交任务的线程执行，实现简单的`feedback`机制
    - `DiscardPolicy` 直接丢弃
    - `DiscardOldestPolicy` 直接丢弃最最前面的任务(可能是优先级最高的任务)

#### 线程池工作逻辑
- 如果`当前线程数<corePoolSize`,  有新任务时将创建1个新线程，即使当前有`idle`线程
- 如果`当前线程数>=corePoolSize`，有新任务时
    - 如果`workQueue`未满则加入队列
    - 如果`workQueue`已满，但是`当前线程数<maximumPoolSize`，则创建1个新线程
    - 如果`workQueue`已满，并且`当前线程数=maximumPoolSize`，
      此情况称为`线程池饱和(saturation)`, 此时则调用`rejectedExecutionHandler`
- 如果`allowCoreThreadTimeOut(true)`，当核心线程的`idle`时长超过`keepAliveTime`，线程将终止
    - 当线程池引用计数为0，并且没有工作线程，线程池将自动关闭，进而允许JVM退出
    - 不要设置`corePoolSize=0`以达到以上效果，否则必须等待`workQueue`满了才会创建1个新线程
- 常用工作队列
    - `direct handoff` 如`SynchronousQueue`
        - 调用线程直接将工作任务交给工作线程
        - 一般设置`unbounded maximumPoolSize`，避免任务被`rejected`
    - `unbounded` 如`unbounded LinkedBlockingQueue`
        - 因为工作队列不会满，所以线程池仅有核心线程
        - 适用于处理web请求等独立的任务
    - `bounded`   如`ArrayBlockingQueue`
        - 如果配合设置`maximumPoolSize`为固定值，可避免耗尽系统资源
        - 难以调优和控制
    - `PriorityBlockingQueue`
- 当线程池已关闭或者饱和时，提交新任务将触发`rejectedExecutionHandler`
- 线程池里的线程中断后
     - 如果是关闭线程池导致的中断，线程将执行清理任务然后结束
     - 如果是其他原因，线程池(可能)将创建1个新的线程
- 任务不要中断线程池线程，如果捕获了中断异常，应重新抛出或设置中断标志为`true`
- 如果任务抛出任何异常，执行此任务的线程将会结束，线程池(可能)将创建1个新的线程
- 关闭线程池`shutdownNow()`, 将会中断线程池线程以取消正在执行的任务
    - 如果需获取已开始但因关闭而被取消的任务，可覆盖`execute()`，具体见P159 
- 线程池适合执行独立的，同质化(执行时间相当)的任务，以下情况谨慎考虑使用线程池
    -  执行互相依赖的任务
    -  执行`thread confinement`任务，如任务仅能在单线程里执行
    -  执行需要快速响应的任务，线程池里长期任务可能导致响应延迟
    -  执行使用`ThreadLocal`的任务，线程池里线程时共享的，导致数据泄露
- 如果1个任务执行时添加新的任务到同1个线程池，并且等待其执行结果，将造成死锁(`resource deadlock`)


## 任务
- 任务接口:`Runnable`,`Callable`,`Future`
- `FutureTask` -> `Future` ，可视为`Promise`
    - `get()`获取结果,阻塞线程，可能抛出以下错误
        - `CancellationException`  任务取消
        - `ExecutionException`     任务执行出错
        - `InterruptedException`   当前线程中断
        - `TimeoutException`       获取结果超时
    - `cancel()` 取消任务，可选是否中断当前线程
    - `isCancelled()` 是否已经取消
    - `isDone()` 是否已结束，包括取消任务或执行出错
    - `done()/set()/setException()/runAndReset()` Hook方法
- 任务一旦结束，则状态不再改变，除非内部调用`runAndReset()`
- 提交给`Executor`的任务都将封装为`Future`，以便调用者获取结果或取消任务
- 通过以下方式在任务取消时释放资源(如关闭`socket`)
    - 覆盖`Thread.interrupt()`
    - 覆盖`Future.cancel()`
    - 覆盖`ThreadPoolExecutor.newTaskFor()`,自定义封装任务的`Future.canel()`
- `poison pill` 使用特殊的任务，当线程执行此任务时，结束执行并退出

## `Thread.UncaughtExceptionHandler`
- 如果代码没有处理抛出的异常，则此异常将交由`UEH`处理
- `UEH`查找顺序
    - 当前线程
    - 当前线程所属线程组`ThreadGroup`
    - 此线程组的父线程组
    - 当前线程的`DefaultUncaughtExceptionHandler`
    - 如果仍然没有，则输出错误到`System.err`
- 长期运行的线程，应总是设置`UEH`

## 关闭JVM
- `Runtime.getRuntime().addShutdownHook()`
- 如果仅有`daemon`线程，JVM将正常退出
- 正常关闭JVM将将回调hook，直接终止JVM进程或`Runtime.halt()`则不会

## `CompletionService`
- `ExecutorCompletionService` -> `CompletionService`
- `ECS`可视为1个存储任务执行结果的`BlockingQueue`
- 提交给`ECS`的任务将由传入的`Eexcutor`执行
- 每执行完1个任务，将保存到`ECS`内部的`BlockingQueue`
- 调用`ECS`的`take()/poll()`获取执行结果
