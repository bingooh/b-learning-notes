## Discard
- `var Discard io.Writer = devNull(0)`
- 放弃写入的数据，类似输出重定向到`/dev/null`
```
r:=strings.NewReader("hi")
io.Copy(ioutil.Discard,r)
```

## NopCloser()
- `func NopCloser(r io.Reader) io.ReadCloser`
- 封装`r`为`ReadCloser`
- 调用返回的`ReadCloser.Close()`无任何影响

## ReadAll()
- `func ReadAll(r io.Reader) ([]byte, error)`
- 从`r`读取全部数据直到`EOF`或报错
- `err==nil`表示成功读取，err不会等于`EOF`

## ReadDir()/ReadFile()/WriteFile()
- `func ReadDir(dirname string) ([]os.FileInfo, error)`
    - 读取文件目录 
- `func ReadFile(filename string) ([]byte, error)`
    - 读取文件内容
    - `err==nil`表示成功读取，err不会等于`EOF`
- `func WriteFile(filename string, data []byte, perm os.FileMode) error`
    - 把`data`写入文件
    - 如果文件不存在，则创建文件并设置为`perm`权限
    - 如果文件已存在，则先`truncate`再写入

## TempDir()/TempFile
- `func TempDir(dir, prefix string) (name string, err error)`
    - 在`dir`下创建临时目录，目录名称使用`prefix`作为前缀
    - 如果`dir==""`，则使用默认的系统临时目录
    - 并发调用将创建不同的临时目录
    - 调用者负责删除自己创建的目录
- `func TempFile(dir, pattern string) (f *os.File, err error)`
    - 创建临时文件并打开以便进行读写操作
    - 如果`dir==""`，则使用默认的系统临时目录
    - 文件名称使用`pattern`并在结尾或替换`*`为随机字符串
    - 并发调用将创建不同的临时文件
    - 调用者负责关闭及删除自己创建的文件