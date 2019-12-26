## Misc
- `log` 提供1个默认的logger
    - 日志默认输出到`Stderr`
    - 每条日志默认输出为1行
    - 日志格式默认为`log.LstdFlags`
- `log.New()`     创建1个新的logger
- `log.PrintXX()` 打印日志
- `log.PanicXX()` 打印日志，然后调用`panic()`
- `log.FatalXX()` 打印日志，然后调用`os.Exit(1)`
- `log.SetXX()`   设置日志格式(`Flag`)，前缀(`Prefix`)，输出(`Writer`)
```
log.Print(1)

log.SetPrefix(">>>")
log.Print(2)

//输出到Stdout
log.SetOutput(os.Stdout)
log.Print(3)

//输出所在文件，log.SetFlags(0)则只显示传入的日志数据
log.SetFlags(log.Lshortfile|log.LstdFlags)
log.Print(4)

//直接输出日志
log.Output(1,"5")
```