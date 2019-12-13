# go build

## 用法

```
usage: go build [-o output] [-i] [build flags] [packages]
```

## 不带任何参数

`go build`不带任何参数的话，编译当前目录所对应的代码包。

## 带需要编译的文件

测试目录如下，

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
└── src
    └── main
        ├── lib.go
        └── main.go

5 directories, 2 files
[root@localhost]~/code#
```

如果我们需要在`go build`后面带上具体的文件，那么我们需要带上这个package的所有文件。

```shell
[root@localhost]~/code/src/main# ls
lib  lib.go  main.go
[root@localhost]~/code/src/main# go build main.go lib.go
[root@localhost]~/code/src/main#

```

生存的可执行文件的文件名就是`main`。如果我们这样`go build lib.go main.go`，生成的可执行文件名就是`lib`。

## 指定输出的文件名

```shell
[root@localhost]~/code/src/main# go build -o victor
[root@localhost]~/code/src/main# ls
lib.go  main.go  victor
[root@localhost]~/code/src/main#

```

可以使用`-o`选项指定编译得到的文件名。

## 编译包

前面谈到的`go build`编译的都是针对`package`进行编译。

如果我们啥参数都没有给到`go build`命令，那么`go build`就是针对当前目录对应的`package`进行编译。

## 生成不同平台架构下的可执行程序

```shell
[root@localhost]~/code/src/main# GOOS=darwin GOARCH=amd64 go build
[root@localhost]~/code/src/main# ls
lib.go  main  main.go
[root@localhost]~/code/src/main# file main
main: Mach-O 64-bit executable
[root@localhost]~/code/src/main#

```

`GOOS=darwin`指定平台是`MacOSX`，`GOARCH=amd64`则指定处理器架构是`amd64`。

## GO的目录和package可以不一样

Go的目录和package的名字可以不一样，但是一个目录下面只能有一个package。

## go build 附加参数

| 附加参数 | 备注                                        |
| -------- | ------------------------------------------- |
| -v       | 编译时显示包名                              |
| -n       | 打印编译时会用到的所有命令，但不真正执行    |
| -a       | 强制重新构建                                |
| -p n     | 开启并发编译，默认情况下该值为 CPU 逻辑核数 |
| -x       | 打印编译时会用到的所有命令                  |
|          |                                             |

### linkshared

`go build`可以带上一个`-linkshared`参数。

>link against shared libraries previously created with -buildmode=shared.

通过`-linkshared`，我们可以编译得到一个依赖动态库的目标文件。目标文件会比不带`-linkshared`参数编译出的目标文件小很多。

## go build的 build flags

我们知道`go build`的用法是

```
go build [-o output] [-i] [build flags] [packages]
```

这里的build flags  有哪些呢？

### -gcflags

> arguments to pass on each go tool compile invocation.

这样使用

`go build -gcflags="-l -N" -o interface11 interface11.go`

具体有哪些gcflags可供使用，可以这样

`go tool compile`

## golang 静态库使用

我们可以将某个`package`编译成静态库，然后其他代码使用这个静态库提供的一些方法。

TODO：如何知道某个静态库提供了哪些方法呢？

### 静态库的生成

我们有如下目录结构。

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
└── src
    ├── main
    │   └── main.go
    └── services
        └── manage_service.go

6 directories, 2 files
[root@localhost]~/code#
```



其中`main.go`文件如下：

```go
package main

import (
        . "services"
)

type TestNameOff struct {
        Id              int64
        Name    string
}

func main() {
        SaveInstance()
}
```

`manage_service`文件如下：

```go
package services

import "fmt"

func SaveInstance() {
        fmt.Println("Save Instance")
}
```

我们想把`services`包编译成一个静态库。



## buildmode

`go build`可以带上一个`-buildmode`参数，表示`go build`的输出产物是什么。

1. `-buildmode=archive`
2. `-buildmode=default`
3. `-buildmode=shared`
4. `-buildmode=exe`
5. `-buildmode=pie`
6. `-buildmode=plugin`



这些参数分别表示啥含义呢？

### archive

我们将某一个`package`编译成一个静态库（就是linux下面的`.a`文件），提供给其他代码使用。

还是以上面谈的的`services`包为例。

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
└── src
    ├── main
    │   └── main.go
    └── services
        └── manage_service.go

6 directories, 2 files
[root@localhost]~/code#
```

我们想将`services`包编译成一个静态库，怎么做呢？两种方法。

1. `go install`

   ```shell
   [root@localhost]~/code/src/services# ls
   manage_service.go
   [root@localhost]~/code/src/services# go install services
   [root@localhost]~/code/src/services# ls -l ~/code/pkg/linux_amd64
   总用量 16
   -rw-r--r--. 1 root root 12538 12月  3 13:17 services.a
   [root@localhost]~/code/src/services#
   
   
   ```

   直接使用`go install`命令，格式为`go install xxx`，其中`xxx`表示某一个包。

   这样我们就可以在该工程对应的`$GOPATH/pkg`目录找到对应的静态库文件，也就是`services.a`文件

2. `go build -buildmode=archive`

   使用`-buildmode=archive`参数。

   ```shell
   [root@localhost]~/code/src/services# ls
   manage_service.go
   [root@localhost]~/code/src/services# go build -buildmode=archive -o services.a
   [root@localhost]~/code/src/services# ls -l
   总用量 20
   -rw-r--r--. 1 root root    86 12月  2 22:23 manage_service.go
   -rw-r--r--. 1 root root 12538 12月  3 13:19 services.a
   [root@localhost]~/code/src/services#
   
   ```



这两种方法获得的静态库文件是一样的。通过`md5sum`可以看到。

```shell
[root@localhost]~/code/src/services# md5sum services.a
07d4b572b69e8bfb512b7e6bacd1671e  services.a
[root@localhost]~/code/src/services# md5sum ~/code/pkg/linux_amd64/services.a
07d4b572b69e8bfb512b7e6bacd1671e  /root/code/pkg/linux_amd64/services.a
[root@localhost]~/code/src/services#

```



我们得到了静态库文件，怎么使用呢？

我们目录如下，删除了`services`目录。

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
│   └── linux_amd64
│       └── services.a
└── src
    └── main
        └── main.go

6 directories, 2 files
[root@localhost]~/code#
```

正常来说，`go build`是通不过的。

```shell
[root@localhost]~/code/src/main# go build main
main.go:5:2: cannot find package "services" in any of:
        /usr/local/go/src/services (from $GOROOT)
        /root/gopath/src/services (from $GOPATH)
        /root/code/src/services
[root@localhost]~/code/src/main#
```

原因是`main`包里import了`services`包，但是现在`services`包被我们删除了。

我们可以使用`go tool compile`和`go tool link`来解决这个问题。

```shell
[root@localhost]~/code/src/main# go tool compile -I /root/code/pkg/linux_amd64 main.go
[root@localhost]~/code/src/main# go tool link -o victor -L /root/code/pkg/linux_amd64 main.o
[root@localhost]~/code/src/main# ls -l
总用量 1976
-rw-r--r--. 1 root root     125 12月  3 01:14 main.go
-rw-r--r--. 1 root root    6248 12月  3 13:25 main.o
-rwxr-xr-x. 1 root root 2008844 12月  3 13:25 victor
[root@localhost]~/code/src/main# ./victor
Save Instance
[root@localhost]~/code/src/main#
```

`go tool compile`编译得到目标文件`main.o`，`go tool link`链接`main.o`和`services.a`得到可执行文件`victor`。

参考文章：

1. https://reborncodinglife.com/2018/04/27/how-to-create-static-lib-in-golang/

### default

`-buildmode=default`就是默认的模式，输出得到的就是一个可执行文件。`go build`不加任何参数的时候，buildmode就是`default`

### shared

`-buildmode=shard`表示编译产物是`so`文件，也就是动态链接库。



我们还是以下面这个目录结构为例，

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
└── src
    ├── main
    │   └── main.go
    └── services
        └── manage_service.go

6 directories, 2 files
[root@localhost]~/code#
```

我们需要将`services`包编译成`so`文件，然后给`main.go`调用。

> Before compile any shared library, the standard builtin packages should be installed as shared library. This will allow any other shared library to link with them.

在编译共享库（`so`文件）前，我们需要先安装 **标准库** 的动态链接库。

```shell
[root@localhost]~/code# go install -buildmode=shared -linkshared std
[root@localhost]~/code#

```

结果是：

```
[root@localhost]/usr/local/go/pkg/linux_amd64_dynlink# ls
archive            debug               hash             io.shlibname    net.shlibname      regexp.shlibname   syscall.a
bufio.a            encoding            hash.a           libstd.so       os                 runtime            syscall.shlibname
bufio.shlibname    encoding.a          hash.shlibname   log             os.a               runtime.a          testing
bytes.a            encoding.shlibname  html             log.a           os.shlibname       runtime.shlibname  testing.a
bytes.shlibname    errors.a            html.a           log.shlibname   path               sort.a             testing.shlibname
compress           errors.shlibname    html.shlibname   math            path.a             sort.shlibname     text
container          expvar.a            image            math.a          path.shlibname     strconv.a          time.a
context.a          expvar.shlibname    image.a          math.shlibname  plugin.a           strconv.shlibname  time.shlibname
context.shlibname  flag.a              image.shlibname  mime            plugin.shlibname   strings.a          unicode
crypto             flag.shlibname      index            mime.a          reflect.a          strings.shlibname  unicode.a
crypto.a           fmt.a               internal         mime.shlibname  reflect.shlibname  sync               unicode.shlibname
crypto.shlibname   fmt.shlibname       io               net             regexp             sync.a             vendor
database           go                  io.a             net.a           regexp.a           sync.shlibname
[root@localhost]/usr/local/go/pkg/linux_amd64_dynlink# ls -l libstd.so
-rw-r--r--. 1 root root 37738040 12月  3 22:28 libstd.so
[root@localhost]/usr/local/go/pkg/linux_amd64_dynlink# file libstd.so
libstd.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=4d114936c58f67b6583dbda168184c27d1a30d3d, not stripped
[root@localhost]/usr/local/go/pkg/linux_amd64_dynlink#

```

可以看到在`$GOOT/pkg/linux_amd64_dynlink`目录下有`libstd.so`共享库文件。

然后我们就可以编译`services`的动态链接库了。

```
[root@localhost]~/code/src/services# ls
manage_service.go
[root@localhost]~/code/src/services# go install -buildmode=shared -linkshared services
[root@localhost]~/code/src/services# ls -l ~/code/pkg/linux_amd64_dynlink
总用量 2028
-rw-r--r--. 1 root root 2055920 12月  4 12:52 libservices.so
-rw-r--r--. 1 root root   12680 12月  4 12:52 services.a
-rw-r--r--. 1 root root      15 12月  4 12:52 services.shlibname
[root@localhost]~/code/src/services#

```

可以看到生成了`libservices.so`文件以及`services.a`文件。

现在目录如下：

```shell
[root@localhost]~/code# tree
.
├── bin
├── lib
├── pkg
│   └── linux_amd64_dynlink
│       ├── libservices.so
│       ├── services.a
│       └── services.shlibname
└── src
    ├── main
    │   ├── main
    │   └── main.go
    └── services
        └── manage_service.go

7 directories, 6 files
[root@localhost]~/code#
```

添加参数`-linkshared`编译`main`包，

```shell
[root@localhost]~/code/src/main# ls
main.go
[root@localhost]~/code/src/main# go build -linkshared main
[root@localhost]~/code/src/main# ls -lh
总用量 20K
-rwxr-xr-x. 1 root root 16K 12月  5 00:38 main
-rw-r--r--. 1 root root 126 12月  5 00:08 main.go
[root@localhost]~/code/src/main# ldd main
        linux-vdso.so.1 =>  (0x00007ffc76378000)
        libstd.so => /usr/local/go/pkg/linux_amd64_dynlink/libstd.so (0x00007f52436f9000)
        libservices.so => /usr/local/go/pkg/linux_amd64_dynlink/libservices.so (0x00007f52433f8000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f524302a000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f5242e26000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5242c0a000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f524616d000)
[root@localhost]~/code/src/main#
[root@localhost]~/code/src/main# ./main
Save Instance
[root@localhost]~/code/src/main#
```

可以看到得到的可执行文件`main`只有16K，通过`ldd`查看其依赖的`so`库，可以看到其依赖`libservices.so`，这正是我们之前编译得到的。

**需要注意的是，我们这里编译`main`包的时候，必须保留`services`包的源码在`GOPATH`目录下，才可以编译通过**

有些疑问是，如果我们拿到了第三方提供的`so`文件，还必须要得到源码，才能编译成功么？看下面的参考文章2是这么说的。还是感觉奇怪，和C不一样。C的共享库的话，是不需要提供源码给`so`库使用方的。只需要提供一个头文件即可。

参考文章：

1. http://blog.ralch.com/tutorial/golang-sharing-libraries/
2. http://reborncodinglife.com/2018/04/29/how-to-create-dynamic-lib-in-golang/

### plugin

参考文章：

1. Https://www.jianshu.com/p/448647a279ca