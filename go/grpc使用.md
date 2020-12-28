# 安装

1. 配置代理

   ```shell
   export GOPROXY=https://goproxy.cn
   ```

2. 下载grpc包

   ```shell
   # victordong @ VICTORDONG-MB1 in ~/code/gomodProj [0:29:35]
   $ go get -u google.golang.org/grpc
   
   ```

   具体步骤参看这里，https://pkg.go.dev/google.golang.org/grpc

3. 安装protocol buffer
   grpc不是必须依赖 protocol buffer的，也可以是基于http的。但是我们一般都是使用pb。使用protocol buffer，需要安装protocol buffer compiler，protoc。protoc用于编译`.proto`文件，生成go代码。`.proto`文件包含了service和message的定义。
   先下载protoc的压缩包，

   ```shell
   $ PB_REL="https://github.com/protocolbuffers/protobuf/releases"
   $ curl -LO $PB_REL/download/v3.13.0/protoc-3.13.0-linux-x86_64.zip
   ```

   解压，

   ```shell
   $ unzip protoc-3.13.0-linux-x86_64.zip -d $HOME/.local
   ```

   更新环境变量，

   ```shell
   export PATH="$PATH:$HOME/.local/bin"
   ```

   更详细的步骤参看这里，https://grpc.io/docs/protoc-installation/

   借助protoc，可以把`.proto`文件转译为各种编程语言对应的源码，包含数据类型定义、调用接口等。

   protoc在设计上把protobuf和不同的语言解耦了，底层用c++来实现protobuf结构的存储，然后通过**插件**的形式来生成不同语言的源码。

   可以把protoc的编译过程分成简单的两个步骤，

   - 解析.proto文件，转译成protobuf的原生数据结构在内存中保存
   - 把protobuf相关的数据结构传递给相应语言的编译插件，由插件负责根据接收到的protobuf原生结构，渲染输出特定语言的模板

4. 为protoc安装go的一些plugins
   因为我们都是基于go module的，所以如果你的go版本为1.12或者以下的话，那么这样做，

   ```shell
   $ export GO111MODULE=on  # Enable module mode
   $ go get google.golang.org/protobuf/cmd/protoc-gen-go \
            google.golang.org/grpc/cmd/protoc-gen-go-grpc
   ```

   更新 PATH，

   ```shell
   $ export PATH="$PATH:$(go env GOPATH)/bin"
   ```

   上面2个plugin是干啥用的呢？

   - protoc-gen-go
     The protoc-gen-go binary is a protoc plugin to generate Go code for both proto2 and proto3 versions of the protocol buffer language.
     更多信息可以查看，https://developers.google.com/protocol-buffers/docs/reference/go-generated
   - protoc-gen-go-grpc
     protoc-gen-go-grpc is a plugin for the Google protocol buffer compiler to generate Go code. 
     更多信息查看这里，https://godoc.org/google.golang.org/grpc/cmd/protoc-gen-go-grpc

