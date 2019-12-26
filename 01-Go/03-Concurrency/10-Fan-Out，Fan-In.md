## Misc
- `Fan-Out` 启动多个协程处理同1个管道的数据
- `Fan-In`  接收多个管道的数据并发送到同1个管道
```
func fanOut(
	ctx context.Context) ([]<-chan int) {

	//耗时操作
	longFn := func() int {
		i:=rand.Intn(6)
		fmt.Println("longFn将耗时： ",i)

		time.Sleep(time.Duration(i)* time.Second)

		return rand.Int()
	}

	//执行耗时操作
	run := func(intStream chan<- int, i int) {
		defer close(intStream)

		select {
		case <-ctx.Done():
			return
		case intStream <- longFn():
			return
		}

	}

	//启动多个协程
	count := 5
	streams := make([]<-chan int, count);
	for i := 0; i < count; i++ {
		intStream := make(chan int);
		streams[i] = intStream

		go run(intStream, i)

	}

	return streams
}

func fanIn(
	ctx context.Context,
	upstreams ... <-chan int,
) (<-chan int) {
	var wg sync.WaitGroup
	wg.Add(len(upstreams))

	intStream := make(chan int)
	go func() {
	    //等待所有upstream关闭，然后关闭intStream
		wg.Wait()
		close(intStream)
	}()

	run:= func(upstream <-chan int) {
		defer wg.Done()
		
		//如果upstream为nil，可能造成错误或死锁，以下写法不完善
		for i:=range upstream{
			select {
			case <-ctx.Done():
				return
			case intStream<-i:
			}
		}
	}
    
    //把多个upstream数据输出到intStream
	for _,upstream:=range upstreams{
		go run(upstream)
	}


	return intStream
}

func main() {
    //在10秒后自动取消ctx
	ctx, cancelCtx:= context.WithTimeout(context.Background(), 10*time.Second)

	streams := fanOut(ctx)
	resultStream:=fanIn(ctx,streams...)

	start:=time.Now()
	
	//以下应使用for select，否则可能造成死锁
	for i:=range resultStream{
		fmt.Println("result: ",i)
	}
    
    //对比fanIn耗时，应与最慢的longFn耗时相差不多
	fmt.Println("fanIn耗时：",time.Since(start).Seconds())

	cancelCtx()
}

```