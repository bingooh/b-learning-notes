## Misc
- `sync`包提供低级的内存访问同步的编程基元，如互斥锁，资源池等
- 建议仅在以下情况使用`sync`（见P65）
    - 需要高性能的临界块
    - 守护`struct`的内部状态
    - 不需要与其他函数协调执行的函数

## WaitGroup
- 用于等待多个协程执行结束，即创建`join point`
- 父协程调用`Add()`设置需等待的子协程数
- 父协程启动子协程，然后调用`Wait()`等待子协程执行完成
- 子协程调用`Done()`表示执行完成，每调用1次`Done()`计数+1
- 当调用`Done()`次数与父协程设置的等待数相同时，父协程继续执行
```
var wg sync.WaitGroup

count:=5
wg.Add(5)

//Add()和Done()尽量写在一起
for i := 0; i < count; i++ {
	go func(i int){
		defer wg.Done()
		fmt.Println(i)
	}(i)
}

wg.Wait()
```

## Mutex，RWMutex
- 互斥锁，在任何时间仅能被1个协程获取
- 两者变量值为`nil`表示为`unlock`状态
- 不是`ReentryLock`, 即已获取锁的协程再次获取会造成死锁
- `RWMutex` 读写互斥锁
    - 在任意时间可同时被多个`reader`或者1个`writer`获取
      即如果有1个`writer`获取此锁，则其他`reader`/`writer`不能再获取此锁
    - 适用于读多写少的情况
```
var mu sync.Mutex
mu.Lock()
mu.Lock() //死锁，不能多次获取
```

## Once
- 同1个`once`对象，仅调用1次传入的函数
- 递归调用可能造成死锁
```
var once sync.Once

initA:= func() {fmt.Println("A")}
initB:= func() {fmt.Println("B")}

once.Do(initA)
once.Do(initB)  //不再执行，即使传入另一个函数

var onceA,onceB sync.Once
var initB func()

initA:= func() {onceB.Do(initB)}
initB= func() {onceA.Do(initA)} //再次获取同1把锁，导致死锁

onceA.Do(initA) //死锁
```

## Cond
- 可理解为1种`集合点`，协程会一直等待直到接收到事件通知
- 每个`Cond`都关联1个锁`cond.L`,改变条件或判断条件不满足必须先获取锁
    - 判断条件是否满足指读取共享变量的值，如不满足应调用`Wait()`
    - 改变条件指改变共享变量的值，改变后应调用`Signal()`、`Broadcast()`发送事件通知
    - 综上，必须先获取同1个`cond.L`，然后读/写共享变量
- 协程执行`Wait()`自动释放锁，唤醒后自动获取锁
    - 协程调用`Wait()`前应先获取锁，调用`Wait()`后自动释放锁
    - 协程唤醒后应再次判断条件是否满足(使用`for`循环)
- 唤醒等待中的协程使用`FIFO`算法
    - `Signal()`      唤醒队列里第1个协程
    - `Broadcast()`   唤醒全部协程，没有唤醒顺序(还是唤醒后重新竞争锁?)，效率比管道高
- 基本流程
    - `cond.L.Lock()`
    - 在`for`循环里判断条件是否满足
        - 如满足则调用`cond.Wait()`，即不满足业务条件
        - 判断条件会读共享变量
    - `for`循环结束，即满足业务条件，执行后续操作
        - 执行操作会写共享变量
    - `cond.L.Unlock()`
    - `cond.Signal()`
```
size:=5
queue:=make([]int,size) //共享变量
cd:=sync.NewCond(&sync.Mutex{})

produce:= func() {
	for{
		cd.L.Lock() //获取锁以便操作共享变量queue，调用Wait()

		//在for循环里判断条件，建议for块不要执行其他语句
		for len(queue)>0{
			cd.Wait()
		}

		queue=append(queue,1)
		fmt.Println("添加元素到队尾")

		cd.L.Unlock()
		cd.Signal() //不需要获取锁
	}
}

consume:= func() {
	for{
		cd.L.Lock()

		for len(queue)==0{
			cd.Wait()
		}
        
        //queue[0]=nil //如果queue引用对象，建议置为nil避免内存泄露
		queue=queue[1:]
		fmt.Println("从队首删除元素")

		cd.L.Unlock()
		cd.Signal()
	}
}
```

## Pool
- 适用于存放独立的未使用的临时对象
- 临时对象如果只被对象池引用，则在GC时会被回收
- 使用对象池注意事项
    - `New()`的实现需要线程安全
    - `Get()`返回的对象可能是已使用过的，注意判断其状态
    - 对象使用完成后应调用`PUT()`返回对象池
    - 同1池里的对象应该是独立的，统一类型的(`uniform`)
```
pool:=&sync.Pool{
	New: func() interface{} {
		return struct {}{}
	},
}

pool.Put(pool.New())  //初始化放入对象
pool.Put(pool.Get())  //使用结束后应放回池里
```

## Map
- `Map` 线程安全的`concurrent map`，适用于
    - 读多写少的情况
    - 多协程下，每个协程操作的`key`集合不相交(`disjoint`)

## atomic
- `atomic`包提供原子操作
- `atomic.Value` 可用于配置文件