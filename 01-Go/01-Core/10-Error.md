## Misc
- 按照使用目的，错误可分为：
    - `error` 业务错误，可暴露给用户
    - `panic` 程序错误，但可恢复，如不恢复将退出程序
    - `fatal` 致命错误，不可恢复，应直接退出程序

## error
- `error`是内建类型
- 函数返回`error`：
    - `error`应放在最后1个返回值
    - 返回`nil`表示没有发生错误
- 应在第一时间检查函数返回的`error`
- `error`应视为程序的一种分支状态
```
//处理错误的一般方式
f,err:=os.Create(name)
if err!=nil{
    return err
}
defer f.Close()

//自定义错误
Err1=errors.New("Bad")
Err2:=fmt.Errorf("%v","Bad")

type MyErr struct {
	name,msg string
}

//实现error接口
func (e *MyErr) Error() string{
	return e.name+"\n"+e.msg
}

//使用类型断言获取底层错误对象
var e error
if err,ok:=e.(*MyErr);ok{
	fmt.Println(err.name)
}

```

## panic/recover
- 调用`panic()`后程序将会执行`defer`语句
    - 如果`defer`里有调用`recover()`，程序将终止退出
    - 如果`defer`里没调用`recover()`，程序执行完`defer`后退出
    - 如果在`recover()`里调用`panic()`，程序将继续退出
- 某些情况下程序将自动`panic`，而不是返回`error`,如执行`1/0`
- 建议仅在程序初始化时使用`panic/recover`
```
func parse()([]int,error){
	var o []int

	defer func() {
		if err:=recover();err!=nil{
			o=nil            //如果出错，设置结果为空
			err=err.(MyErr)  //如果err类型是MyErr则继续执行，否则re-panic
		}
	}()

	return doParse(o),nil
}

```