## 基本概念
- `Synchronizer(同步点)` 
    - 封装1个状态以决定线程继续执行还是等待
    - 提供方法以修改封装的状态
- `CountDownLatch`
    - 创建`latch`，设置初始化计数为`N`
    - 调用`await()`，如果`N==0`，则继续执行,否则阻塞线程
    - 调用`countDown()`,`N--`
    - 当N减少到`0`时, `await`的线程被唤醒继续执行
    - 一旦`N==0`，则不再改变(门打开后不再关闭)
- `FutureTask`
    - 调用`get()`将阻塞线程，直到返回结果
    - 可视为初始计数为`1`的`CountDownLatch`
- `Semaphore`
    - `Semaphore`管理`N`个虚拟的许可证
    - 调用`acquire()`获取许可证, 如果`N==0`则阻塞当前线程
    - 调用`release()`释放许可证, 进而唤醒1个阻塞的线程
    - `Semaphore+List`可实现`BlockingQueue`, 可管理有限资源
    - `Semaphore(N==1)`可实现`mutex(互斥锁)`
- `CyclicBarrier`
    - `Barrier`可视为1个线程集合点(`Rendezvous`) 
    - `Latch`等待事件发生,`Barrier`等待线程集合
    - 创建`CyclicBarrier`对象, 设置要等待的线程数(`parties`)`N`
    - 调用`await()`，内部计数+1
    - `内部计数==N`时,
        - 内部计数清0
        - 如果有则执行`barrierAction`
        - 释放所有线程继续执行，进入下一轮集合
    - `barrierAction`指每次全部集合后，在集合点执行的操作
- `Exchanger`
    - 适用于2个线程互相交互数据，可视为1个双向的`SynchronousQueue`

## 高效的可伸缩的对象缓存
```
public class Cache{
    //提供需要缓存的值，可能比较耗时
    private final Supplier<String> supplier;
    
    //存储缓存值
    private final ConcurrentHashMap<String,Future<String>> map=...;
    
    public Cache(Supplier<String> supplier){
        this.supplier=supplier;
    }
    
    //获取缓存值
    public String get(String key){
        //未使用线程同步块
        while(true){
            Future<String> f=map.get(key);
            if(f==null){
                //可能创建多个task，但是还未执行supplier，所以不影响性能
                FutureTask<String> task=new FutureTask(this.supplier);
                
                //如果返回null，说明key还未存放过task，即首次存放
                //如果存在，说明是另1个线程放入的
                //此处利用ConcurrentHashMap做了1次线程同步
                f=map.putIfAbsent(key,task);
                
                //首次存放，则执行task，进而调用supplier
                if(f==null){f=task;task.run();}
            }
            
            try{
                //如果任务还未执行完成，将阻塞线程
                return f.get();
            }catch(CancellationException e){
                //取消任务，则移除缓存
                map.remove(key,f);
            }
        }
    }
}

//使用Cahce
private final Cache cache=new Cache(
    ()->{...//获取要缓存的值，耗时计算}
);

//在多线程环境下调用
String v=cache.get(i);

```
