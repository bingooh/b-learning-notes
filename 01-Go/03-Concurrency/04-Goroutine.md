## Misc
- `goroutine` 一个运行中的并发执行的函数
- `go` 创建1个新的`goroutine`
- `main goroutine` 每个Go程序都具有的主协程
- `goroutine`可称为协程`coroutine`（本笔记不加区别）:
    - 两者都是比线程更高1级的抽象，不允许中断 
    - 协程没有优先级(`nonpreemptive`)，`goroutine`阻塞时为低优先级
    - 协程允许自行暂停，继续执行。`goroutine`阻塞时自动暂定，解除阻塞时自动继续执行
-  `goroutine`是轻量级的
    - 1个新启动的协程仅需要几KB
    - 协程的上下文切换消耗很少系统资源
- `M:N scheduler` Go管理协程的机制
    - 映射`M green threads(程序管理的线程)`到`N OS threads`
    - 分布协程到`green threads`，保证协程阻塞时，其他协程可正常运行
- Go采用`fork-join`并发模型
    - 父协程`fork`N个并发执行的子协程
    - 子协程最终(执行完成后)会重新加入父协程，其加入点称为`join point`
    - 父协程执行完成后，全部子协程自动销毁，不管子协程是否有执行
    - 一般父协程需要提供`join point`以等待子协程重新加入

#### join point
```
var wg sync.WaitGroup
wg.Add(1)

//fork
go func(){
    defer wg.Done()
    fmt.Println("hi")
}()

wg.Wait()  //join point


c:=make(chan int);

//fork 
go func() {
	time.Sleep(1*time.Second)
	c<-1
}()

i:=<-c  //join point
fmt.Println(i);
```

#### variable scope
```
var wg sync.WaitGroup

count:=5
wg.Add(5)

//此写法只输出4
for i := 0; i < count; i++ {
	go func(){
	    defer wg.Done()
		fmt.Println(i)
	}()
}

//以下两种写法正常输出
for i := 0; i < len; i++ {
	go func(k){
	    defer wg.Done()
		fmt.Println(k)
	}(i) //使用参数
}

for i := 0; i < len; i++ {
    i:=i //使用局部变量，屏蔽了for块的变量i
	go func(){
	    defer wg.Done()
		fmt.Println(i)
	}()
}

wg.Wait()

```

## GOMAXPROCS
- `scheduler`决定如何执行协程，包括3个主要概念
    - `G` 1个协程
    - `M` 1个系统线程，也称为`machine`
    - `P` 1个执行上下文，也称为`processor`，一般实现为`Deque(双端队列)`
- `scheduler` 执行过程
    - 创建`GOMAXPROCS`指定数量的`双端队列(P)`，即任务队列
    - 创建对应数量的`线程M`，保证每个任务队列绑定1个线程用于执行任务
    - 每启动1个新`协程G`，会有相应的任务加入队列尾部，线程从队列里取出任务并执行
    - `GOMAXPROCS`默认与CPU核心数相同，调试时可增大此设置以便快速重现死锁错误
- 线程执行任务过程
    - 首先从自己的任务队列尾部取出任务并执行
    - 如果自己的任务队列为空，则从其他任务队列头部获取任务，即`work-stealing`
    - 如果任务因执行IO等操作导致线程被阻塞
        - 从线程池取出1个空闲线程负责执行被阻塞线程的任务队列
        - 被阻塞的线程不再与当前任务队列关联，但仍然与当前任务关联
        - 当线程不在阻塞时，首先尝试获取1个队列，并将当前任务加入
        - 如果不能获取队列，则将任务加入全局队列`global context`,而线程则放入线程池
        - 任意线程的队列为空时，会优先从全局队列获取任务，并且也会定时获取全局队列的任务
- 综上所述
    - `GOMAXPROCS`设置任务队列数`P`，而不是线程数`M` 
    - Go运行时使用线程池，并且保证每个队列有1个线程负责执行
    - 任务`G`实际为`协程的当前状态`，Go实现的`work-stealing`算法实际操作的是`continuation`,见P297