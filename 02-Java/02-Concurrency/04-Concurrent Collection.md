## 基本概念
- 同步集合使用`Concurrent`，避免使用`synchronize`可能导致的性能降低问题
- 大部分情况下可使用`ConcurrentXXX`替代`SynchronizedXXX`
- `ConcurrentXXX`在单线程环境下有少许的性能损耗

## Concurrent Collection/Map
- `CoucurrentMap`
    - `CoucurrentHashMap`
    - `CoucurrentSkipListMap` ->`SortMap`
- `Collection`
    - `CopyOnWriteArrayList` 适合多读少写 
    - `CoucurrentSkipListSet` -> `SortSet`
- `Queue` ->`Collection`
    - `ConcurrentLinkedQueue`
    - `PriorityQueue`
- `Deque(双端队列)` ->`Queue`  
    - `ConcurrentLinkedDeque`

## `ConcurrentHashMap`
- 使用`lock striping`,允许多线程并发访问
- 提供`putIfAbsent()`等方法
- 使用`iterator`不会触发`CME`
- 因为支持并发,会有以下缺点
    - 创建`iterator`后,另1线程删除了元素,
      `iterator`仍然可以访问到删除的元素
    - `size()/isEmpty()` 只能返回模糊的结果
    - 不支持使用任何锁执行同步操作

## `CopyOnWriteArrayList`
- 适用于多读少写情况,比如用于存储事件监听器
- 内部使用数组存储元素,在写操作时,同步复制原有数组并添加写入的新元素


## `BlockingQueue`
- `BlockingQueue`分为`bounded`/`unbounded`
- `BlockingQueue`的阻塞
    - 如果`queue`已满, 阻塞添加操作
    - 如果`queue`已空, 阻塞获取操作
    - `unbounded queue` 不阻塞添加操作, 不断添加可能导致内存耗光
- `Queue`提供两类方法
    - `add()   /remove() /element()` 如果失败抛出错误
    - `offer() /poll()   /peek()`    如果失败返回`null`/`false`
- `BlockingQueue` ->`Queue` 提供两类阻塞方法
    - `put()/take()` 一直阻塞,直到操作完成
    - `offer (timeout)/poll(timeout)` 阻塞操作,超时返回`null`/`false`
- `BlockingQueue`适合实现`Producer/Consumer`，或者`CSP`
    - 分隔`Producer/Consumer`的代码实现
    - 推荐优先使用`bounded queue`,实现`throttle`
    - `P/C`模式实现了`serial thread confinement`
      1个可变对象从`P`线程传递给`C`线程
- `Deque` 双端队列,适合实现`work stealing`
    - 每个`worker`线程有自己的任务`deque`
    - 如果某个`worker`的队列空了,则从其他worker的队列尾部取任务执行
    - 网络爬虫,通常worker同时为`P/C`,`worker`发现新页面URL,会封装为
      任务添加到自己或其他`worker`的任务队列里
- 常见`BlockingQueue`
    - `ArrayBlockingQueue`
    - `LinkedBlockingQueue`
    - `PriorityBlockingQueue` 根据优先级对元素排序
    - `SynchronousQueue`      不缓存任何元素，效率更高
    - `DelayQueue`            指定延迟后才能获取元素
- 常见`Deque`
    - `ArrayDeque`
    - `LinkedList`
    - `LinkedBlockingDeque`
