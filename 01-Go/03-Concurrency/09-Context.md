## Misc
- `Context` 类似`pipeline`节点传递的`done <-chan xx`，不同点：
    - `Context`可以设置超时，截止日期等，满足条件后自动取消
    - `Context`可以关联数据，以便在多个节点间传递
- `Context`接口包含以下方法
    - `Done()` 返回`Done`管道，取消`Context`将导致此管道关闭 
    - `Err()`  返回错误以说明取消原因
        - 如果`Done`还未关闭，则返回`nil`
        - 如果`Done`已经关闭，则返回：
            - `context.Canceled`           说明是主动取消
            - `context.DeadlineExceeded`   说明是超时或超过截止时间而取消
    - `Deadline()` 返回设置的截止时间，超过截止时间将取消`Context`
    - `Value()` 设置或获取与`Context`关联的数据
- `Context`是只读的，每个`Context`对象应派生于1个父`Context`对象
    - `Background()` 返回1个空的根`Context`，不能取消
    - `TODO()`       同上，但主要用于工具分析以确定`Context`对象在程序间正确传递
    - `WithCancel()`    派生的`Context`需主动调用返回的`cancel()`取消
    - `WithDeadline()`  派生的`Context`超过截止时间自动取消
    - `WithTimeout()`   派生的`Context`超时自动取消
- `Context`是并发安全的，取消父`Context`将取消所有派生的`Context`
- `Context`使用守则
    - `Context`应作为节点函数的第1个参数传递，便于工具分析
    - 不要使用`nil Context`，如不确定使用`TODO()`
    - 节点函数应使用派生的`Context`定制取消策略
    - Context仅应用于传递`request-scoped`数据，不要传递可选数据
```
func main() {
	var wg sync.WaitGroup

	rootCtx:=context.Background()
	ctx1,cancel1:=context.WithCancel(rootCtx)
	ctx2,_:=context.WithTimeout(rootCtx,2*time.Second)

	deadline:=time.Now().Add(3*time.Second)
	ctx3,_:=context.WithDeadline(rootCtx,deadline)

	time.AfterFunc(1*time.Second,cancel1)

	print:= func(ctx context.Context,tag string) {
		defer wg.Done()
		<-ctx.Done()

		fmt.Println(tag," canceled")
		fmt.Println(tag,"is DeadlineExceeded: ",ctx.Err()==context.DeadlineExceeded)
	}

	wg.Add(3)
	go print(ctx1,"ctx1")
	go print(ctx2,"ctx2")
	go print(ctx3,"ctx3")

	wg.Wait()
}
```