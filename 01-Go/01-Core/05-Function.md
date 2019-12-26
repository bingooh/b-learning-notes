## 函数声明
- Go函数是`first-class`
- 函数支持变长参数，返回多个值
- 函数传参或变量赋值时使用`复制`,包括复制指针地址值
```
func hi(){}
func hi(i,j int,s string){}
func hi(i int,)

func hi(s ...string){}    //变长参数
hi([]string{"a","b"}...)  //展开参数

func hi() int{return 1}
func hi2() (i int){i=1;return}         //使用命名返回参数
func hi3() (int,string){return 1,"a"}  //返回多个值

func hi(fn func() int){}  //fn作为参数

var fn func() int         //fn作为变量
fn:=func() int{return 1}  //匿名函数赋值给变量

func(){}()  //直接执行匿名函数

//闭包
func hi(i int) func(){
	return func(){
		i++
	}
}

```



## defer
- `defer`用于延迟执行语句
- `defer`后面的语句将在函数返回前执行
- `defer`的语句按照`后进先出`的顺序执行
- 调用`os.Exit()`程序立刻退出，不会执行`defer`语句
```
//输出 2 1 0
for i:=0;i<3;i++{
	defer fmt.Printf("% v",i);
}

```