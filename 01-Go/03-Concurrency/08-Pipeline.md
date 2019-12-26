## Misc
- `pipeline`非常类似面向函数编程的`pipe()`
- `pipeline`的每个节点称为`stage`
    - 当前节点接收上一级节点的输出作为输入
    - 当前节点的输出将作为下一节点的输入
    - 上级节点可取消下级节点的执行
    - 第1个节点负责产生数据，称为`generator`
    - 综上，每个节点可视为1个函数
- `pipeline`函数签名：
```
func(
    done <-chan xx,
    upstream <-chan xx,
    args...,
)(downstream <-chan xx){}
```

```
func repeatFn(
	done <-chan int,
	fn func() int) (<-chan int) {

	intStream := make(chan int)

	go func() {
		defer close(intStream)

		for {
			select {
			case <-done:
				return
			case intStream <- fn():
			}
		}
	}()

	return intStream
}

func take(
	done <-chan int,
	upstream <-chan int,
	count int) (<-chan int) {

	intStream := make(chan int)

	go func() {
		defer close(intStream)

		for i := 0; i < count; i++ {
			select {
			case <-done:
				return
			case intStream <- <-upstream:
			}
		}
	}()

	return intStream
}

func main() {
	done:=make(chan int)
	defer close(done)

	fn:= func() int{return rand.Intn(100)}
	intStream:=take(done,repeatFn(done,fn),10);

	for i:=range intStream{
		fmt.Println(i)
	}

}
```