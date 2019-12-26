## 格式化代码
- `gofmt -w -l -s xxx` 格式化代码并保存要源文件
- `go fmt xxx` 格式化代码，底层使用`go fmt`
- 建议使用`tab`缩进

## 代码注释
- `/**/`(块注释),`//`(行注释)
- 注释使用空白行换行
- 函数注释的首个单词应与函数名称相同
- 包注释的首个单词应为`Package`
- 包注释如果太多，可写到`doc.go`
- 包注释仅需添加到任意1个相同包名的源文件，
  如果添加到多个，生成的注释文档将按照源文件名称排序
- `godoc` 提取注释并生成帮助文档
- `godoc xxpkg | grep -i xx` 查找注释
- `godoc -http=:6060 -play` 开启本地文档服务器，访问对应地址可查看代码注释

## 语法格式(自动插入分号)
- 编译器自动在每个语句末尾插入分号 
> if the newline comes after a token that could end a statement, insert a semicolon
- `{`不要单独写一行，这也是Go唯一要求的语法格式
```
//以下代码将报错
map1:=map[string]string{
    "1":"a"
}

//以上代码等价于
map1:=map[string]string{
    "1":"a"; //自动插入分号，报错
}

//使用以下方式避免报错
map1:=map[string]string{
    "1":"a",
}

map1:=map[string]string{
    "1":"a"}
```

## 程序入口
- `main`包的`main()`为程序入口
- 任何源文件都可包含`init()`以初始化
- 程序执行顺序：
    - 初始化变量
    - 初始化导入包
    - 如果有执行`init()`
    - 执行`main()`

## 字符&编码
- 1个16进制数占4个bit，2个16进制数占1个byte
- 字符集即整数到字符的映射，这里的整数称为`Code Point`
- 大部分字符集的前128个字符兼容ASCII字符集
- `ASCII，Unicode`是字符集
    - `Unicode`  2~3字节表示1字符，最大为`0x10FFFF`
    - `ASCII`    7bit表示1字符，仅收录了128字符
    - `ISO-8859` 1字节表示1字符，包含多个系列
        - `ISO-8859-1` 收录西欧字符,也称为`Latin1`
        - `ISO-8859-5` 收录斯拉夫字符，如俄语
- 如果将`ASCII`码转换为`Unicode`码并保存，英文文档体积将增加1倍
- `UTF8/UTF16/UTF32`是`Unicode`的一种实现方式，决定如何存储`UCP(Unicode Code Point)`
- `UTF8`采用变长编码方式,对`UCP`重新编码
    - `U+0000~U+007F`   使用1个字节
    - `U+0080~U+07FF`   使用2个字节
    - `U+0800~U+FFFF`   使用3个字节，中文一般在此范围里
    - `U+10000~U+1FFFF` 使用4个字节，1个`UCP`最多占4个字节
- `UTF16`采用2~4字节编码1个`UCP`,使用`BOM(byte order marker)`区分字节序
    - `U+0000~U+FFFF`   使用2个字节
    - `U+10000~U+1FFFF` 使用4个字节
    - `BOM`占2个字节，添加在`string`变量的最前面
    - `little endian(BOM==FEFF)` 低字节序，低字节前，高字节后
    - `big    endian(BOM==FFFE)` 高字节序，高字节前，低字节后
- Go`字符串字面量`使用`UTF8`编码,`rune`表示1个字符，即1个`UCP`
    - 编码：`UCP(rune)`转换为`UTF8([]byte)`
    - 解码：`UTF8([]byte)`转换为`UCP(rune)`
    - 编译器将`字符串字面量`转换为`UTF8`字节数组，存储到`string`变量
    - `string`变量可存储任意字节，如存储客户端传入的`GBK`编码的字节数组