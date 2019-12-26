## 参考文档
- [grpc quick start](https://grpc.io/docs/quickstart/go/)
- [protobuf release](https://github.com/google/protobuf/releases)

## Misc
- `google/protobuf/descriptor.proto`  定义元数据(`.proto`结构)
- 序列化输出多个消息到同1个`流`时，需自行区分消息边界
- 建议使用多个独立消息组成的大数据集，1个独立消息建议小于1MB
- 默认序列化输出不包含`.proto`文件内容
    - 必须有`.proto`才能正确解析序列化数据
    - 支持把`.proto`输出到序列化数据，即动态消息

## 安装pb/grpc
- 安装protoc，下载并解压安装包，并设置环境变量
- 安装protoc-gen-go
    - `go get -u github.com/golang/protobuf/protoc-gen-go`
- 安装grpc
    - `go get -u google.golang.org/grpc`
- 安装Goland插件`Protobuf Support`

## 代码生成
- `protoc --proto_path=XX --xx_out=DST_DIR xx.proto`
    - `--proto_path` proto文件查找路径，等价于`-I XX`，多个则重复此参数
    - `-xx_out`      生成代码输出的目录/压缩文件,`xx`为代码语言，如`go`
    - `xx.proto`     proto文件路径,多个空格分隔。必须位于`--proto_path`
- `--xx_out`实际设置`protoc`使用的插件
    - `--xx_out=p1=v1,p2=v2:DST_DIR`  设置插件参数和结果输出路径
    - `--go_out=plugins=grpc:DST_DIR` 设置生成Go&gRPC代码