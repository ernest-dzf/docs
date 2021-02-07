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
   grpc不是必须依赖 protocol buffer的，也可以是基于http的。但是我们一般都是使用pb。使用protocol buffer，需要安装protocol buffer compiler，protoc。protoc用于编译`.proto`文件，与其他插件（比如protoc-gen-go）配合，生成go代码。
   对于使用grpc来说，`.proto`文件一般都包含了service和message的定义。
   先下载protoc的压缩包（针对的是linux，如果是mac，请下载mac对应的压缩包，比如`[protoc-3.14.0-osx-x86_64.zip](https://github.com/protocolbuffers/protobuf/releases/download/v3.14.0/protoc-3.14.0-osx-x86_64.zip)`），

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

   安装完成之后，

   ```shell
   # victordong @ VICTORDONG-MB1 in ~ [1:04:57]
   $ protoc --version
   libprotoc 3.14.0
   
   # victordong @ VICTORDONG-MB1 in ~ [1:05:00]
   $
   ```

   借助protoc，可以把`.proto`文件转译为各种编程语言对应的源码，包含数据类型定义、调用接口等。

   protoc在设计上把protobuf和不同的语言解耦了，底层用c++来实现protobuf结构的存储，然后通过**插件**的形式来生成不同语言的源码。

   ![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/protoc_gen_go.png)

   可以把protoc的编译过程分成简单的两个步骤，

   - 解析.proto文件，转译成protobuf的原生数据结构在内存中保存
   - 把protobuf相关的数据结构传递给相应语言的编译插件，由插件负责根据接收到的protobuf原生结构，渲染输出特定语言的模板

4. 为protoc安装go的一些plugins
   因为我们都是基于go module的，所以如果你的go版本为1.12或者以下的话（不能太低，否则都不支持gomodule），那么这样做，

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

5. 这时候可以查看一下`protoc`，`protoc-gen-go`，`protoc-gen-go-grpc`是否都已经安装完成。

   ```shell
   
   # victordong @ VICTORDONG-MB1 in ~/code/playground [2:08:44]
   $ protoc --version
   libprotoc 3.14.0
   
   # victordong @ VICTORDONG-MB1 in ~/code/playground [2:08:48]
   $ protoc-gen-go --version
   protoc-gen-go v1.25.0
   
   # victordong @ VICTORDONG-MB1 in ~/code/playground [2:08:54]
   $ protoc-gen-go-grpc --version
   protoc-gen-go-grpc 1.0.1
   
   # victordong @ VICTORDONG-MB1 in ~/code/playground [2:09:01]
   $
   ```

6. 关于`protoc-gen-go`和`protoc-gen-go-grpc`的区别。
   前面说了，`protoc-gen-go`是用于生成`.proto`文件对应的go代码的。注意这里我根本没有谈到grpc，和grpc没关系。就算不使用grpc，有时候我们还是会使用protobuf，那么就使用`protoc-gen-go`就好。protobuf和grpc是两个项目，本质上没有啥关系。
   关于protobuf，有好几个相关的包，我们该使用哪个呢？

   - `github.com/golang/protobuf`
   - `github.com/protocolbuffers/protobuf-go`
   - `google.golang.org/protobuf`

   对于`github.com/golang/protobuf`，这个package诞生很早，2010年就诞生了，那时候GO 1还没发布。

   > We call the original version of Go protocol buffers APIv1, and the new one APIv2. Because APIv2 is not backwards compatible with APIv1, we need to use different module paths for each.
   >
   
   `github.com/golang/protobuf`就是所谓的 APIv1，`google.golang.org/protobuf`就是所谓的APIv2。两者是不兼容的，如果是新业务代码，我们都建议使用APIv2，就是 google.golang 开头的那个。
   
   那`github.com/protocolbuffers/protobuf-go`又是什么鬼?
   
   这个其实就是 `google.golang.org/protobuf`这个module在github上的一个仓库，你可以认为两者是一样的。但是你使用的时候，还是建议import `google.golang.org/protobuf` 这个。
   
   从 v1.20 开始，`google.golang.org/protobuf`包中的`protoc-gen-go`就不再支持生成 grpc 服务代码，转而有1个单独的项目，叫做`protoc-gen-go-grpc`。
   
   > The v1.20 [`protoc-gen-go`](https://pkg.go.dev/google.golang.org/protobuf/cmd/protoc-gen-go) does not support generating gRPC service definitions. In the future, gRPC service generation will be supported by a new `protoc-gen-go-grpc` plugin provided by the Go gRPC project.
   
   也就是说，如果你使用新版本的APIv2（`google.golang.org/protobuf`），那么还需要配合使用 protoc-gen-go-grpc，才能生成grpc相关代码。
   
   但是对于`github.com/golang/protobuf`， `protoc-gen-go`还是支持grpc的。
   
   > The `github.com/golang/protobuf` version of `protoc-gen-go` continues to support gRPC and will continue to do so for the foreseeable future.
   
   参考：
   
   - https://blog.golang.org/protobuf-apiv2，讲了开发APIv2的初衷
   - https://blog.golang.org/protobuf，讲了第一版 APIv1的发布
   - https://github.com/protocolbuffers/protobuf-go，这是APIv2版本在github上的仓库
   - https://github.com/protocolbuffers/protobuf-go/releases/tag/v1.20.0，讲了新版本对grpc的支持情况
   - https://grpc.io/docs/protoc-installation/#binary-install，讲了protoc的安装
   - https://grpc.io/docs/languages/go/quickstart/#prerequisites，讲了如果使用grpc的话，你需要做的一些前期工作