## Misc
- `bytes`   操作字节切片
- `strings` 操作UTF8编码字符串
- 两者提供的函数基本相同，仅函数参数不同(`[]byte/string`)
- `[]byte/string`互相转换底层会进行复制，因此提供2套函数

## Reader
- 封装字节切片/字符串为只读流
- 实现io接口`Reader/ReaderAt/Seeker/WriterTo/ByteScanner/RuneScanner`
```
// 传入的字节切片对br来说是只读的
// size表示总字节数，len表示剩余可读字节数
br:=bytes.NewReader([]byte("你好"))
fmt.Printf("size:%v, len:%v \n",br.Size(),br.Len()) //size:6, len:6

//读取1字节，剩余5字节可读,也可使用br.ReadByte()
br.Read(make([]byte,1))
fmt.Printf("size:%v, len:%v \n",br.Size(),br.Len()) //size:6, len:5

//取消读取1字节，剩余6字节可读
br.UnreadByte()
fmt.Printf("size:%v, len:%v \n",br.Size(),br.Len()) //size:6, len:6

//读取1个字符占3字节，剩余3字节可读
r,n,_:=br.ReadRune()
fmt.Printf("char:%c, len:%v\n",r,n)                 //char:你, len:3
fmt.Printf("size:%v, len:%v \n",br.Size(),br.Len()) //size:6, len:3

//读取位置重新定位到开始位置，剩余6字节可读
//也可使用br.UnreadRune()/br.Reset()
br.Seek(0,io.SeekStart)
fmt.Printf("size:%v, len:%v \n",br.Size(),br.Len()) //size:6, len:6

//输出到stdout
br.WriteTo(os.Stdout)

//strings.Reader类似，只是传入字符串
sr:=strings.NewReader("你好")
```

## bytes.Buffer
- 可变长度字节缓存，实现`Reader/Writer`等io接口
- `Buffer` 封装字节切片/字符串为流
- `bufio`  封装普通流为缓存流
```
// 以下性能测试结果基本相同,切片都会自动扩展底层数组
// Buffer更多用于封装为流
bs:=[]byte("hi")
slice:=make([]byte,0)
slice=append(slice,bs...)

buffer:=new(bytes.Buffer)
buffer.Write(bs)

// 底层缓存使用默认大小,64字节?
var bf bytes.Buffer
bf:=new(bytes.Buffer)

// 指定底层缓存大小或设置初始值
// 外部不应使用传入的buf参数
buf:=make([]byte,512)
b:=bytes.NewBuffer(buf)

// 处理字符串
b:=bytes.NewBufferString("line1\nline2\n")
for{
	line,err:=b.ReadString('\n')
	fmt.Print(line) //末尾包括换行符

	if err==io.EOF{break}
	if err!=nil{log.Fatal(err)}
}
```

## strings.Builder
- 拼接字符串，封装为写入流

## strings.Replacer
- 替换字符串，仅提供`WriteString()`