# redis入门

## 安装

下载地址，https://redis.io/download。

下载之后得到

```shell
# root @ vm1 in ~ [17:38:32]
$ ls -l redis-6.2.1.tar.gz
-rw-r--r--. 1 root root 2438367 3月   6 17:34 redis-6.2.1.tar.gz

# root @ vm1 in ~ [17:38:34]
$
```

解压，

```shell
# root @ vm1 in ~/redis-6.2.1 [17:38:56]
$ ls
00-RELEASENOTES  CONDUCT       COPYING  INSTALL   MANIFESTO  redis.conf  runtest-cluster    runtest-sentinel  src    TLS.md
BUGS             CONTRIBUTING  deps     Makefile  README.md  runtest     runtest-moduleapi  sentinel.conf     tests  utils

# root @ vm1 in ~/redis-6.2.1 [17:38:57]
$

```

在`redis-6.2.1`目录下`make`即可，然后就会在src目录下得到一些可执行文件，

```shell
# root @ vm1 in ~/redis-6.2.1/src [17:41:04]
$ find . -type f -executable
./mkreleasehdr.sh
./redis-trib.rb
./redis-server
./redis-sentinel
./redis-cli
./redis-benchmark
./redis-check-rdb
./redis-check-aof

# root @ vm1 in ~/redis-6.2.1/src [17:41:18]
$
```

然后就是启动redis，

```shell
# root @ vm1 in ~/redis-6.2.1/src [12:22:38]
$ nohup ./redis-server ../redis.conf &
.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.2.1 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 5997
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

5997:M 06 Mar 2021 17:43:07.101 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
5997:M 06 Mar 2021 17:43:07.101 # Server initialized
5997:M 06 Mar 2021 17:43:07.101 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
5997:M 06 Mar 2021 17:43:07.102 * Ready to accept connections

```

默认非daemon的方式来运行，且默认服务端口为6379。

客户端登录redis，

```shell
# root @ vm1 in ~/redis-6.2.1/src [17:44:23]
$ ./redis-cli
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379>
```

客户端可以直接`shut down`命令将服务端关闭，

客户端，

```
127.0.0.1:6379> shutdown
not connected>

```

服务端，

```shell
6068:M 06 Mar 2021 17:46:30.510 # User requested shutdown...
6068:M 06 Mar 2021 17:46:30.510 * Saving the final RDB snapshot before exiting.
6068:M 06 Mar 2021 17:46:30.512 * DB saved on disk
6068:M 06 Mar 2021 17:46:30.512 * Removing the pid file.
6068:M 06 Mar 2021 17:46:30.512 # Redis is now ready to exit, bye bye...

# root @ vm1 in ~/redis-6.2.1/src [17:46:30]
$
```

如果需要远程登录，可以这样，

```shell
# victordong @ VICTORDONG-MB1 in ~/redis-6.2.1/src [0:11:08]
$ ./redis-cli -h vm1 -p 6379
vm1:6379>
```

不过如果想要远程登录，记得改下redis启动时候的配置文件监听地址，

```shell
# root @ vm1 in ~/redis-6.2.1 [0:12:40]
$ cat redis.conf|grep bind|grep -v "#"
bind 0.0.0.0

# root @ vm1 in ~/redis-6.2.1 [0:13:00]
$

```

记得监听地址需要为`0.0.0.0`，而不能是`127.0.0.1`这样的。

## redis权限

redis没有类似mysql那样丰富的权限管理。

对于redis-6以下的版本，可以再配置文件`redis.conf`中，增加`requirepass`选项，

```shell
# root @ vm1 in ~/redis-6.2.1/src [0:21:24]
$ cat ../redis.conf|grep requirepass|grep -v "#"
requirepass foobared

# root @ vm1 in ~/redis-6.2.1/src [0:21:35]
$
```

只有密码，没有用户。这里的密码就是`foobared`。

这种就是redis-6以下的配置方式。redis-6以上可以兼容这种配置。如果配置了密码，如果不指定密码的话，那么就会出现鉴权错误，

```powershell
# victordong @ VICTORDONG-MB1 in ~/redis-6.2.1/src [0:21:13]
$ ./redis-cli -h vm1 -p 6379
vm1:6379> get p
(error) NOAUTH Authentication required.
vm1:6379>
```

我们可以这样，

```shell
# victordong @ VICTORDONG-MB1 in ~/redis-6.2.1/src [0:24:09]
$ ./redis-cli -h vm1 -p 6379 -a foobared
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
vm1:6379> get p
"1"
vm1:6379>
```

直接在命令行加`-a`选项。或者，

```shell
# victordong @ VICTORDONG-MB1 in ~/redis-6.2.1/src [0:24:46]
$ ./redis-cli -h vm1 -p 6379
vm1:6379> get p
(error) NOAUTH Authentication required.
vm1:6379> auth foobared
OK
vm1:6379> get p
"1"
vm1:6379>

```

通过`auth`命令鉴权。

这种只有密码的方式，其实是有1个默认的用户名的，叫`default`。

```shell
# victordong @ VICTORDONG-MB1 in ~/redis-6.2.1/src [0:25:53]
$ ./redis-cli -h vm1 -p 6379
vm1:6379> auth default foobared
OK
vm1:6379> get p
"1"
vm1:6379> auth default22222 foobared
(error) WRONGPASS invalid username-password pair or user is disabled.
vm1:6379> get p
"1"
vm1:6379>
```

可以看到，用户/密码是`default/foobared`，只不过这里`default`被省略了。

对于redis-6以上的版本来说，建议使用带有用户名的鉴权方式，即`username/passwd`。

关于鉴权，可以参考官方，https://redis.io/topics/acl，有很详细的说明。

## redis数据结构

## redis常用命令

- setnx key value，不存在就插入，这是老版本的命令，最新版本通过set命令，配合一些option就能做到。setnx可能以后会被废弃。
- del key，删除key