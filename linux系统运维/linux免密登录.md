# 免密登录 #
## ssh的登录方式 ##
1. 客户端连接上服务器之后，服务器把自己的公钥传给客户端
2. 客户端输入服务器密码通过公钥加密之后传给服务器
3. 服务器根据自己的私钥解密登录密码，如果正确那么就让客户端登录

## 公钥认证 ##
1. 客户端即A端生成RSA公钥和私钥

	一般在用户的根目录新建一个.ssh/.文件夹，在文件夹中通过ssh-keygen -t rsa命令来产生一组公私钥。

	如下图所示id_rsa为私钥，id_rsa.pub为公钥。
2. 客户端将自己的公钥存放到服务器：在生成了公私钥之后要实现AB两端的交互认证，这两个文件肯定不能只放到A端，当然也需要在B端（服务器端）做一下登记，我们自己（A端）保留自己的私钥，然后把公钥id_rsa.pub存放到B端。

	在服务器端（B端）的.ssh/目录下还会有authorized_keys+know_hosts，这两个文件。

	authorized_keys:存放远程免密登录的公钥,主要通过这个文件记录多台机器的公钥，上面提到的A端在生成自己的公私钥之后，将公钥追加到authorized_keys文件后面。

	know_hosts : 已知的主机公钥清单，这个作为A端和B端都会自动生成这个文件，每次和远端的服务器进行一次免密码ssh连接之后就会在这个文件的最后追加对方主机的信息（不重复）

### 配置免密登录方式

比如有三台机器，vm1，vm2和vm3。

首先分别登录三台机器，执行`ssh-keygen -t rsa`，生成秘钥。这样在三台机器的`.ssh`目录下就会有秘钥文件。

```shell
# root @ localhost in ~/.ssh [0:42:32]
$ ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts

# root @ localhost in ~/.ssh [0:42:34]
$
```

然后在vm1执行，

```
cat id_rsa.pub >> authorized_keys
scp authorized_keys root@vm2:~/.ssh/
```

在vm2执行，

```shell
cat id_rsa.pub >> authorized_keys
scp authorized_keys root@vm3:~/.ssh/
```

在vm3执行，

```shell
cat id_rsa.pub >> authorized_keys
scp authorized_keys root@vm1:~/.ssh/
```

在vm1执行，

```shell
scp authorized_keys root@vm2:~/.ssh/
```

这样，三台机器vm1、vm2、vm3的authorized_keys文件都有各自的公钥了。