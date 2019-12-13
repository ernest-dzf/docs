# go开发环境搭建

- 下载安装包，https://studygolang.com/dl。

- 解压到`/usr/local/`目录

  ```shell
  [root@vm3 go]# pwd
  /usr/local/go
  [root@vm3 go]# ls
  api              doc          PATENTS     test
  AUTHORS          favicon.ico  pkg         VERSION
  bin              lib          README.md
  CONTRIBUTING.md  LICENSE      robots.txt
  CONTRIBUTORS     misc         src
  [root@vm3 go]# 
  
  
  ```

- 配置环境变量

  ```
  export GOROOT=/usr/local/go
  export GOPATH=/root/gopath
  export GOBIN=$GOPATH/bin
  export PATH=$PATH:$GOROOT/bin
  export PATH=$PATH:$GOPATH/bin
  ```

  配置`/etc/profile`环境变量，source一下。

- 验证

  ```shell
  [root@vm3 gopath]# go version
  go version go1.12.9 linux/amd64
  [root@vm3 gopath]# 	
  ```

  
  
# gvm安装

使用gvm可以方便地在多个不同的go版本之间切换。

```shell
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

```

由于可能被墙，建议使用shadowsocks + privoxy 搭建http代理。