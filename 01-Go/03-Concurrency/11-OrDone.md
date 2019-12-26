## Misc
- `orDone`指传入的任何1个管道关闭，则返回的管道关闭
```
func orDone(
	ctx context.Context,
	upstream <-chan int,
) (<-chan int) {

	intStream := make(chan int)

	go func() {
		defer close(intStream)

		for {
			select {
			case <-ctx.Done():
				return
			case i, ok := <-upstream:
				if ok == false {
					return
				}

				select {
				case <-ctx.Done():
				case intStream <- i:
				}
			}
		}
	}()

	return intStream

}

func main() {
	ctx,cancel:=context.WithTimeout(context.Background(),2*time.Second)

	upstream:=make(chan int)
	go func() {
		upstream<-1
		upstream<-2

		close(upstream)
	}()

	for i:=range orDone(ctx,upstream){
		fmt.Println(i)
		time.Sleep(500*time.Millisecond)
	}

	cancel()
}

```