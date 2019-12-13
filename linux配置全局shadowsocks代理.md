# linux全局配置http代理

利用sslocal搭建socks5代理，利用privoxy搭建http转socks代理。

## 安装privoxy

privoxy可以将socks5代理转成http代理。

1. 安装epel源

   ```shell
   yum install epel-release
   ```

2. 安装privoxy

   ```shell
   yum info privoxy
   yum install privoxy
   ```

3. 修改配置
   改之前先保存一下旧配置。

   ```shell
   cp  /etc/privoxy/config /etc/privoxy/config.bak
   ```

   编译配置文件

   ```shell
   vim /etc/privoxy/config
   ```

   搜索关键字`listen-address`， 找到 `listen-address 127.0.0.1:8118` 这一句，保证这一句没有注释，8118 就是将来 http 代理所使用的端口。
   然后搜索 `forward-socks5t`, 将 `forward-socks5t / 127.0.0.1:1080 .` 此句的注释去掉.

   注意这里的1080端口，就是我们后面`sslocal`服务起的端口号。

4. 启动

   ```shell
   systemctl start privoxy	
   ```

5. 配置代理
   执行`vim /etc/profile`，添加下面，

   ```shell
   export http_proxy=http://127.0.0.1:8118
   export https_proxy=http://127.0.0.1:8118
   export ftp_proxy=http://127.0.0.1:8118
   ```

   然后source一下。这样我们所有的http请求都是走的代理。

## shadowsocks

这个是在我们已经有了shadowsocks的服务端的情况下的教程。

安装shadowsocks，

```shell
yum install epel-release -y
yum install python-pip
pip install --upgrade pip
pip install shadowsocks
```

启动`sslocal`，

```
/usr/bin/sslocal -c /etc/shadowsocks.json
```

```shell
[root@localhost]~/code/src/main# cat /etc/shadowsocks.json
{
    "server":"129.226.65.145",
    ##目标ssserver ip地址
    "server_port":48799,
    ##目标ssserver port
    "local_address": "127.0.0.1",
    "local_port":1080,
    ##本地sslocal服务监听地址
    "password":"PASSWORD",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
}
[root@localhost]~/code/src/main#
```

如果想将sslocal放入开机启动，

```
echo "nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &" >> /etc/rc.local 
```

privoxy也可以像上面这样加入到`rc.local`里面。