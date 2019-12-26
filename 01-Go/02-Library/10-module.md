### 设置代理
```
# 设置Git代理
git config --global http.proxy=http://xxx:xx

# 设置Go代理，Go从环境变量http_proxy获取代理地址
# Windows可直接设置环境变量，也可以使用以下命令别名
# 也可以设置GOPROXY环境变量
proxyon=set http_proxy=http://127.0.0.1:1080
proxyoff=set http_proxy=
```

### go package
- `go help packages` 包帮助文档
- 包导入路径
    - `. / ..`  匹配文件系统目录
    - `xxx`     匹配`$GOPATH/src/xxx`
    - `xxx/...` 匹配`$GOPATH/src/xxx`及其子目录
- 特殊包名称
    - `main` 主包，对应编译为1个可执行程序
    - `all`  `$GOPATH`或`main module`下所有包
    - `std`  所有Go标准库包，可使用`go list std`查看
    - `cmd`  所有Go命令包
- 参考文档
    - [Standard Go Project Layout](https://github.com/golang-standards/project-layout) 

### go get
- `go get xxpkg/@version` 下载并安装包，版本默认为`@latest`
- `go get xxx/...`        下载并安装包及其所有子目录
- `go get -d` 仅下载不安装包
- `go get -v` 显示详细信息
- `go get -v` 下载测试依赖包
- `go get -u`       更新到最新`minor`版本
- `go get -u=patch` 更新到最新`patch`版本

### GOPATH & Vendor
- GO1.5支持从`vendor`目录查找依赖包(需设置环境变量，后续不用)
- GO1.5查找依赖顺序(启用`vendor`)
    - `./vendor`
    - `.../verdor` 不断向上查找  
    - `$GOPATH/src/vendor`
    - `$GOPATH`
    - `$GOROOT`
- 使用建议
    - `main package`    仅在根目录下包含1个`vendor`目录 
    - `library package` 不应包含任何`vendor`目录
- 参考文档
    - [Vendor Directories](https://golang.org/cmd/go/#hdr-Vendor_Directories)
    - [govendor](https://github.com/kardianos/govendor)

### GO Module
- Go1.11支持使用`module`管理依赖，但必须设置环境变量`$GO111MODULE`
    - `auto` 如果当前包目录包含`go.mod`并且不在`$GOPATH`下则使用`module`，否则使用`$GOPATH`
    - `on`  使用`module`管理依赖
    - `off` 使用`$GOPATH`管理依赖
- GO1.12默认设置`GO111MODULE=auto`
- GO1.13直接使用`module`，废除`$GOPATH`
- `module`帮助命令
    - `go help modules` 模块帮助文档
    - `go help go.mod`  依赖文件帮助
    - `go help mod`   
    - `go mod`
- `go.mod`
    - 此文件定义当前模块信息，如模块名称，依赖模块
    - 执行Go命令可能会修改此文件设置依赖，如`go get`
    - `go mod init xxx/xxpkg` 初始化模块信息，创建`go.mod`
    - `go mod download / go get ./...` 下载模块依赖
    - `-mod=readonly` 禁止部分Go命令修改此文件
    - `$GOPATH/pkg/mod` 依赖模块存储目录
- `main module`
    - 执行Go命令会从当前目录不断向上查找`go.mod` 
    - 最先找到的包含`go.mod`目录称为主模块目录
    - 模块目录可以存放在任意位置，不要求存放到
    - `go list -m `     打印主模块包名称        
    - `go list -m all`  打印主模块编译列表
    - `go list -m -f={{.Dir}}` 打印主模块根目录路径
- 模块版本
    - 推荐使用`semantic version`，如`v1.0.0` 
    - 版本设置支持使用`@tag,@commit-hash,@branch` 
    - 下载依赖时版本将解析为`vx.x.x-yyyymmddhhmmss-commit-hash`
- `gopkg.in`
    - 声明`module gopkg.in/yaml.v2`
    - 导入`import gopkg.in/yaml.v2`
    - 文档`https://gopkg.in/yaml.v2`
    - 实际链接`github.com/go-yaml/yaml`
    - 注：`v2`一般为分支
- 模块编译
    - 主模块的直接和间接依赖模块将添加到编译列表`build list`
    - 如果依赖模块版本冲突，则取最新版本
    - `-mod=vendor` 从主模块的`vendor`目录查找依赖，默认忽略
    - `go mod vendor` 将主模块所有依赖打包到`vendor`目录
- 参考文档
    - [gopkg.in](http://labix.org/gopkg.in) 
    - [Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)
    - [Use Modules on local filesystem](https://github.com/golang/go/wiki/Modules#can-i-work-entirely-outside-of-vcs-on-my-local-filesystem)
