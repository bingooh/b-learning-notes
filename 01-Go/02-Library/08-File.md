## path/filepath
- `path`      处理以`/`作为分隔符的路径，如URL
- `filepath`  处理文件路径，路径分隔符与系统相关。如Windows路径分隔符为`\\`
    - `EvalSymlinks(name)` 获取软连接文件连接的目标文件路径 

## FileMode
- 定义文件类型和文件权限，即Linux的文件模式，如`0777` 
- 标准库提供多个`FileMode`常量，如`ModeDir,ModePerm`
- `FileMode`方法：
    - `IsDir()`     是否为目录
    - `isRegular()` 是否为普通文件
    - `Perm()`      获取文件读取权限
```
//如果rs==1则为软连接文件
fileInfo,_:=os.Stat(".")
rs:=fileInfo.Mode()&os.ModeSymlink
```

## FileInfo
- 文件信息描述，部分方法：
    - `Name()/Size()/ModTime()`   文件名称/大小/修改时间
    - `Mode() FileMode` 通过返回的`FileMode`可判断是否为文件目录等
    - `isDir()`         底层调用`Mode().isDir()`
- `os.Stat(name string) (FileInfo, error)`
    - 获取指定文件的`FileInfo` 
    - 如果出错,`error==*PathError`
- `os.Lstat(name string) (FileInfo, error)`
    - 获取指定`symbolic(软连接)` 文件的`FileInfo`，其他同`Stat()`
- `file.Stat()` 获取`file`对象的`FileInfo`

## File
- 已打开的文件描述符`file descriptor`
- 文件的I/O操作必须通过文件描述符

## 打开文件
- 打开文件其实是创建1个文件描述符，然后封装为`*File`
- 打开的文件读写完成后，应调用`file.Close()`关闭
- `OpenFile(name string, flag int, perm FileMode) (*File, error)`
    - 打开文件，如果成功可对文件进行读写操作，如果出错`error==*PathError`
    - `flag` 设置打开方式，如果目标文件不支持指定的打开方式，将会返回错误 
    - `perm` 设置文件模式(`未遮罩umask`)，创建文件时必填
- `Open(name string) (*File, error)`
    - 等价于`OpenFile(name, O_RDONLY, 0)` 
    - 以只读方式打开文件
- `Create(name string) (*File, error)`
    - 等价于`OpenFile(name, O_RDWR|O_CREATE|O_TRUNC, 0666)`
    - 以读写方式打开文件
    - 如果文件不存在则创建，如果存在则截断已有内容
- `NewFile(fd uintptr, name string) *File`
    - 使用`fd`作为文件描述符创建`*File`
    - 如果`fd`不是有效的文件描述符则返回`nil`
    - `os.Stdin/Stdout/Stderr`调用此方法创建
- 打开错误判断
    - 如果打开出错则返回`*PathError`,通过此错误可获取
        - `Op/Path` 操作名称(如`open`)/目标文件路径
        - `Err`     具体的错误，可与`os`定义的错误常量比较
    - `os`提供以下方法判断打开错误
        - `IsExist(err)`         文件已存在
        - `IsNotExist(error)`    文件不存在
        - `IsPermission(error)`  文件不支持指定的打开方式或设置的权限
        - `IsTimeout(error)`     文件读写超时，说明见`file.SetDeadline()`
- 打开方式(`flag`)常量
    - `O_RDONLY/O_WRONLY/O_RDWR` 只读/只写/读写
    - `O_APPEND/O_TRUNC`         附加写入/截断已有内容
    - `O_CREATE`                 如果文件不存在则创建
    - `O_EXCL`                   要创建的文件必须不存在，需与`O_CREATE`一起使用
    - `O_SYNC`                   使用`synchronous I/O`打开
```
file,err:=os.OpenFile("xx",os.O_WRONLY,0)
if os.IsNotExist(err){
	log.Fatal("文件不存在")
}
	
if os.IsPermission(err){
	log.Fatal("文件不支持写入")
}
	
defer file.Close()
```

## 读写文件
- `File`实现的读写接口方法
    - `Read()/ReadAt()`
    - `Write()/WriteAt()/WriteString()`
    - `Close()/Seek()`
- `Truncate(n)`
    - 截断文件内容，只保留`n`字节
    - 如果文件内容小于`n`字节，则填充空字节占位
    - 此方法不影响当前的I/O读写位置
- `Sync()`
    - 把当前系统缓存的文件内容写入持久化存储
    - 文件系统会缓存需要写入文件的内容，后续批量写入以提高效率
- `Chdir()`
    - 改变当前工作目录为当前文件的路径
    - 工作目录指程序执行时所在的根目录
    - 此方法要求当前文件对象必须是目录
- `Chmod/Chown()` 改变文件模式和所有者
- `SetDeadline(t)/SetReadDeadline(t)/SetWriteDeadline(t)`
    - 设置读写截止时间
    - 超过截止时间后，已有或后续的读写操作将返回`Timeout`错误？
    - 超过截止时间后，调用以上方法将返回`Timeout`错误(`PathError`实现了`Timeout`接口)？
    - 大部分文件不支持设置截止时间，调用以上方法将返回错误
    - 管道文件支持设置截止时间，如`net.Conn`
    - 截止时间是1个绝对时间值，可多次调用以上方法设置新的截止时间
- `Readdir(n int) ([]FileInfo, error)`
    - 读取当前文件所在目录下的子目录，使用`Lstat()` 
    - 如果`n>0`，则最多读取`n`个目录。如果读取不到任何子目录，返回错误
    - 如果`n<=0`,则读取全部子目录。如果读取时报错则返回错误和已读取的目录
- `Readdirnames(n int) (names []string, err error)`
    - 同`Readdir()`，返回子目录名称

## 操作文件
- `os`提供以下帮助方法
- `Chdir()` 改变程序工作目录
- `Chmod()` 改变文件模式
- `Chown()/Lchown()` 改变文件所有者
- `Chtimes()`   改变文件访问/修改时间
- `Rename()`    重命名或移动文件，如出错返回`*LinkError`
- `Remove()`    删除文件或空目录
- `RemoveAll()` 递归删除所有文件和目录
- `Link()`      创建硬连接文件
- `SymLink()`   创建软连接文件
- `Readlink()`  获取软连接文件连接的目标文件路径
- `SameFile()`  是否为同一文件，Linux下文件的`inode`相同即为同1个文件
- `Mkdir()/MkdirAll` 创建目录/递归创建目录
- `TempDir()`        获取系统临时目录
- `UserCacheDir()`   获取用户缓存目录
- `IsPathSeparator()`是否为文件路径分隔符
- `Pipe() (r *File, w *File, err error)`
    - 返回1对文件管道，不是管道文件 
    