## Future
- `Future`表示1个异步任务，通过`Future`可获取异步任务执行结果，取消/结束任务
- `Future`一般需要提交给`Executor(线程池)`执行
- 一般使用`Future`的实现类`FutureTask`

#### `Future`方法
- `get(timeout)`   获取结果，阻塞当前线程
    - 如果执行出错，  抛出`ExecutionException`
    - 如果任务被取消，抛出`CancellationException`
    - 如果当前线程被中断，抛出`InterruptedException`
- `cancel()`       取消任务
    - 如果任务已结束，则返回false
    - 如果任务还未开始，则取消后不会执行任务
    - 如果任务已经开始，根据输入参数`mayInterruptIfRunning`决定是否中断线程
        - 如果参数为`false`，任务将继续执行
        - 如果参数为`true`，将调用执行任务线程的`interrupt()`抛出中断错误，业务代码可捕获此错误执行取消逻辑
- `isDone()`       任务是否结束。取消任务，任务抛出异常都将结束任务
- `isCancelled()`  任务是否取消

举例：
```
ExecutorService executor=Executors.newCachedThreadPool();
Future<Integer> future=executor.submit(
    ()->{
        Thread.sleep(1000);//模仿异步任务
        return 1;
    }
);

//获取执行结果，将阻塞当前线程
Integer rs=future.get();
```

## CompletableFuture(CF)
- `CF`继承`Future`,更加类似`Promise`
- 可组合多个`CF`实现业务逻辑

#### `CF`方法：
- `complete(v)` 结束任务并设置返回值,`Promise.resolve()`
- `completeExceptionally(ex)` 结束任务并设置需要抛出的异常,`Promise.reject()`
- `xxx(fn)` 使用默认的`ForkJoinPool`
- `xxxAsync(fn,executor)` 使用自定义的线程池
- `join()` 同`get()`,但是仅抛出`unchecked exception`
- `thenApply(fn)`      当前`CF`值作为`fn`输入参数，`fn`返回值被封装为另1个`CF`
- `thenCompose(fn)`    同上,`fn`返回值必须为另1个`CF`
- `thenCombine(cf,fn)` 同时执行`cf`,两个`CF`值将作为`fn`输入参数
- `thenAccept()` 处理`CF`的值
- `allOf()` 等待所有任务完成，返回1个`CF<Void>`
- `anyOf()` 等待任何1个任务完成，返回`CF<Object>`

#### 性能优化
- 如果执行没有线程阻塞的计算任务，使用Stream默认的`ForkJoinPool`
- 如果执行有线程阻塞的任务，使用自定义的`Executor`结合`CF`

#### 线程池的推荐线程数计算公式
`N=Nc*U*(1+W/C)`
- `Nc` 本机CPU核心数, `Runtime.getRuntime().availabelProcessors()`
- `U`  CPU利用率，介于0~1
- `W`  线程等待时间(阻塞时间)
- `C`  线程运行时间

注：为了避免耗尽系统资源，执行I/O阻塞任务，会设置1个最大线程数

## 举例
- 基本用法
```
CF<Integer> future=new CF<>();
new Thread(()->{
    try{
        future.complete(findTotal());
    }catch(e){
        future.completeExceptionally(e);
    }
}).start();

//等价于以上代码，使用默认的ForkJoinPool
CF<Integer> cf1=CF.supplyAsync(
    ()->{
        return findTotal();
    }
);

//获取返回值，阻塞当前线程
Integer rs=future.get();
```

- 结合Stream执行异步查询
```
//shop.getPrice()需要查询远程服务器，阻塞当前线程
//执行多个具有I/O阻塞的异步任务，需要使用自定义的线程池
ThreadFactory factory=(Runnable r)->{
    Thread t=new Thread(r);
    
    //设置为守护进程，避免线程池的线程阻止JVM退出
    t.setDaemon(true);
    return t;
}

int size=Math.min(shops.size(),100);//取较小的数作为线程数
ExecutorService executor=Executors.newFixedThreadPool(size,factory);

//将shop.getPrice()封装为多个任务并提交到线程池执行
List<CF<Integer>> priceFutures=shops.stream()
    .map(shop->CF.supplyAsync(shop::getPrice,executor))
    .collect(toList());

//获取执行结果，当前线程顺序调用join()获取执行结果
//不能和上面代码写为同一个执行链，否则根据流的特性仍然为顺序执行异步任务
//这里存在2个问题
//1. 任何1个任务出错则整个结果出错
//2. 执行时间取决于最慢的任务
priceFutures.stream()
    .map(CF::join)
    .collect(toList());
```

- 操作多个CF
```
CF<Void>[] cfs=shops.stream()
    .map(shop->CF.supplyAsync(shop::getPrice,executor))
    .map(future->future.thenApply(Discount::count)) //输入价格计算折扣
    .map(future->future.thenCombine(
        CF.supplyAsync(()->TaxService::getTax),  //获取税率，并行执行，使用同1个executor
        (discount,tax)->discount*tax             //计算税收
    ))
    .map(future->f.thenAccept(System.out::println)) //实时读取结果
    .toArray(size->new CF[size]); //收集到数组

//等待所有异步任务执行完成
CF.allOf(cfs).join();
```

