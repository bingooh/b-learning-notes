### Misc
- `Wall Clock` 显示时钟
    - 显示人类可读的时间
    - 操作系统可设置此时钟的时间
- `Monotonic Clock` 单调时钟
    - 用于测量时间(间隔)
    - 时间单调递增，不可设置，不可直接读
    - 仅与当前进程有关，某些系统休眠时可能停止计时
- `time.Time` 
    - `Time` 对象可能仅包含其中1个时钟的时间(可打印查看) 
    - `Now()`返回的`Time`对象同时包含以上2个时钟的时间
    - `Add()`将同时修改2个时钟的时间
    - `Round(0)` 删除单调时间
    - 计算时间间隔时，优先使用单调时钟(需两者都包含此时钟)
    - 解析/格式化/截断/时区转换等显示相关的操作使用显示时钟

### Duration
- 表示两个时间点的间隔纳秒数(`int64`，最大`290年`)
```
v:=10 //不同类型不能计算，需先转型
d1:=time.Duration(v)*time.Minute
fmt.Println(d1) // 10m0s

d2,_:=time.ParseDuration("10m")
fmt.Println(d2) // 10m0s

unit:=int64(time.Minute/time.Second)
fmt.Println(unit) // 60 时间单位
```

### Location
- 表示时区，默认提供`time.UTC/time.Local`
```
//创建UTC+0800时区
loc1:=time.FixedZone("UTC+8",8*60*60)
loc2:=time.FixedZone("UTC+8",int(8*time.Hour.Seconds()))

//操作系统必须有时区数据文件，否则报错
loc3,err:=time.LoadLocation("Asia/Shanghai")
```

### Date
- `Date()`  指定日期创建`Time`
- `Month`   常量，`January==1`
- `Weekday` 常量，`Sunday==0`

### Parse/Format
- `time.RFC3339` JSON日期格式(`layout`)，更多格式见官方文档
- `Unix()/t.Unix()/t.UnixNano()` Unix时间转换
- `Parse(layout,value)` 解析日期字符串
    - 时区默认使用`time.UTC`
    - 如果`value`包含时区，则计算时区偏移时使用`time.Local`
- `ParseInLocation(layout,value,location)` 解析日期字符串,使用指定`location`指定的时区计算时区偏移
- `t.Format()` 转换为日期字符串

### Time
- `==` 比较日期
    - 如果为同1个`Time`对象，返回true
    - 如果2个`Time`对象的时区，日期值相同，则返回true
- `Equal()` 比较日期，先转换为同1时区再比较
- `Truncate()` 截断日期
- `Round()` 四舍五入日期
```
//比较日期
loc:=time.FixedZone("UTC+8",8*60*60)

t1:=time.Date(2000,1,1,0,0,0,0,time.UTC)
t2:=time.Date(2000,1,1,0,0,0,0,time.UTC)
t3:=time.Date(2000,1,1,8,0,0,0,loc)

fmt.Println(t1==t2)  //true
fmt.Println(t1==t3) //false
fmt.Println(t1.Equal(t3)) //true
```

### Ticker/Timer
- `Ticker` 间隔触发事件，类似`setInterval()`
- `Timer`  超时触发事件，类似`setTimeout()`
```
var wg sync.WaitGroup
wg.Add(5)

log:= func() {
	defer wg.Done()
	fmt.Println(time.Now().Second())
}

go func() {
	ok:= func() {fmt.Println("ok")}
	time.AfterFunc(2*time.Second,ok)
}()

go func() {
	fmt.Println(<-time.After(3*time.Second))
}()

go func() {
	for range time.Tick(1*time.Second)  {log()}
}()

wg.Wait()
```
