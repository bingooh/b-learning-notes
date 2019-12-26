## Misc
- `channel`可用于内存同步，但更多用于协程交换数据
- `channel`读取/写入数据使用`FIFO`算法
- 建议管道变量使用`xxxStream`命名
- `channel` 基本操作
    - `make(chan int)` 创建管道
    - `close(ch)`      关闭管道
    - `i,ok:=<-ch`/`ch<-i` 读取/写入数据
    - `i:=range ch`        读取数据直到管道关闭
- 单向、双向管道
    - `chan`(双向), `chan<-`(只写), `<-chan`(只读)    
    - 双向管道可隐式的转换为单向管道
    - 单向管道一般用于方法参数和返回值类型，定义数据有效性
- 缓冲管道`buffered channel`
    - `make(chan int,10)` 创建缓冲容量为`10`的管道
    - `make(chan int)`    创建缓冲容量为`0`的管道
    - 读取数据时，如果管道的当前已缓冲值为`0`，则被阻塞
    - 写入数据时，如果管道的当前已缓冲值等于缓存容量，则被阻塞
    - 如果已经有协程等待读取数据，则写入的数据将直接传给读取者
- `nil channel`
    - 声明时未赋值的管道
    - 读取和写入将被阻塞，关闭此管道将抛出`panic`
- `closed channel`
    - 读取获取默认值`zero value`，永不阻塞
    - 写入或再次关闭抛出`panic`

## 编程思想
- 确定管道的所有者和消费者
- 管道所有者职责
    - 初始化管道
    - 写入数据，或者把管道所有权传给其他协程
    - 关闭管道
    - 封装以上功能，对外仅提供只读管道
- 管道消费者职责
    - 判断管道是否已关闭
    - 处理管道阻塞
    - 读取数据

```
//管道所有者,返回只读管道
chanOwner:= func() <-chan int {
	size:=5

	ch:=make(chan int, size)
	go func() {
		defer close(ch)  //关闭管道代码应尽可能靠近创建管道代码

		for i:=0;i<size;i++{
			ch<-i
		}
	}()

	return ch  //隐式转换为只读管道
}

//管道消费者，即主协程
for i:=range chanOwner(){
	fmt.Println(i)
}

```

## select
- `select`绑定多个管道，管道绑定多个协程
- `select`执行过程
    - 同时判断满足条件的`case`(注：`switch`是顺序判断)
    - 如果有多个满足条件，随机选取1个并执行（Go保证每个被选取的机会是相同的）
    - 如果没有任何1个满足条件，则被阻塞或执行`default`语句(如果有提供)
```
//使用超时避免没有任何满足条件的管道，即避免一直阻塞(死锁)
var ch chan int

select {
case <-ch: //读取nil管道
case <-time.After(1*time.Second):
	fmt.Println("timeout")
}

//设置默认条件避免当前循环没有满足条件的管道
var c1,c2 chan int
start:=time.Now()

select {
case <-c1: //读取nil管道
case <-c2: //读取nil管道
default :
	fmt.Printf("after: %v",time.Since(start))
}

```

## for-select
- 主要用于无限循环执行操作，直到收到终止信号
- 终止信号可通过关闭`done`管道发送
```
for {
    select {
    case <-done:
        return
    case intStream<-i: //写入数据     
    }
}

for {
    select {
    case <-done:
        return
    default: 
        //执行业务操作
    }
    
    //也可以在此处执行业务操作
}
```