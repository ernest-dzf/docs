# Docker简介

虚拟机属于虚拟化技术，而Docker这样的容器技术，也是虚拟化技术，属于**轻量级的虚拟化**。

虚拟机虽然可以隔离出很多“子电脑”，但占用空间更大，启动更慢，虚拟机软件可能还要花钱（例如VMWare）。

容器技术恰好没有这些缺点。它不需要虚拟出整个操作系统，只需要虚拟一个小规模的环境（类似“沙箱”）。

Docker容器启动时间很快，几秒钟就能完成。而且，它对资源的利用率很高（一台主机可以同时运行几千个Docker容器）。此外，它占的空间很小，虚拟机一般要几GB到几十GB的空间，而容器只需要MB级甚至KB级。

以下是对比：

​	

| 特性       | 虚拟机           | 容器             |
| ---------- | ---------------- | ---------------- |
| 隔离级别   | 操作系统级       | 进程级           |
| 隔离策略   | Hypervisor       | CGroups          |
| 系统资源   | 5~15%            | 0~5%             |
| 启动时间   | 分钟级           | 秒级             |
| 镜像存储   | GB~TB            | KB~MB            |
| 集群规模   | 上百             | 上万             |
| 高可用策略 | 备份、容灾、迁移 | 弹性、负载、动态 |



**Docker**本身并不是容器，它是创建容器的工具，是应用容器引擎。



Docker口号：

1. Build, Ship and Run
2. Build once, Run anywhere



Docker技术三大核心概念：

1. 镜像（Image）
2. 容器（Container）
3. 仓库（Repository）



Docker镜像，Docker容器，Docker仓库。



Docker镜像，是一个特殊的文件系统。它除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（例如环境变量）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

利用Docker镜像，我们可以创建一个Docker容器。

我们怎么管理这些Docker镜像呢？

负责对Docker镜像进行管理的，是**Docker Registry服务**。

## Linux容器

Linux 容器（Linux Containers，缩写为 LXC）。

**Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。**

在正常进程的外面套了一个[保护层](https://opensource.com/article/18/1/history-low-level-container-runtimes)。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

## 安装docker

### 使用docker仓库安装

1. 卸载旧的版本

   ```shell
   $ sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

   

2. 安装Docker Engine-Community，使用仓库安装。

   - 设置仓库。

     安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

     ```shell
     $ sudo yum install -y yum-utils \
       device-mapper-persistent-data \
       lvm2
     ```
     
   - 设置源
   
     ```shell
     $ sudo yum-config-manager \
         --add-repo \
         http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
     ```
     
   - 安装 Docker Engine-Community
   
     ```shell
     $ sudo yum install docker-ce docker-ce-cli containerd.io
     ```
   
     
   
   - Docker 安装完默认未启动。并且已经创建好 docker 用户组，但该用户组下没有用户。运行docker命令需要root或者sudo，如果你不想这么干，可以将你的用户加入到docker用户组。`sudo usermod -aG docker $USER;newgrp docker`。
   
   - 启动
   
     ```shell
     systemctl start docker
     ```

 安装指南可以参考官方教程，[这里](https://docs.docker.com/engine/install/centos/#install-from-a-package)

### containerd.io、docker-ce-cli、docker-ce分别是干什么的

- containerd.io - daemon to interface with the OS API (in this case, LXC - Linux Containers), essentially decouples Docker from the OS, also provides container services for non-Docker container managers
- docker-ce - Docker daemon, this is the part that does all the management work, requires the other two on Linux
- docker-ce-cli - CLI tools to control the daemon, you can install them on their own if you want to control a remote Docker daemon

可以参看[这里](https://www.reddit.com/r/docker/comments/dsr6y2/containerdio_vs_dockercecli_vs_dockerce_what_are/)。

### 给docker配置国内源

```shell
$ vim /etc/docker/daemon.json
```

增加如下内容，

```json
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

然后重启，

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## Docker Daemon

用于管理镜像和容器，客户端发起`docker run`的请求会交给docker daemon处理，docker daemon会给这个镜像开辟一个新的容器。

再比如客户端发起`docker pull`命令也是交给docker daemon，docker daemon去检查本地仓库有没有，本地仓库没有的话docker daemon去远程仓库拉取镜像放到本地仓库。

## 使用docker

Docker 是服务器 - 客户端架构。命令行运行`docker`命令的时候，需要本机有 Docker 服务。如果这项服务没有启动，可以用下面的命令启动。

```shell
systemctl start docker
```

### image文件

**Docker 把应用程序及其依赖，打包在 image 文件里面。**

只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。

image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

为了方便共享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库 [Docker Hub](https://hub.docker.com/) 是最重要、最常用的 image 仓库。此外，出售自己制作的 image 文件也是可以的。

### 运行image文件

我们使用`docker image pull xxxx`，获取image文件`xxxx`后，就可以运行这个image。使用`docker container run`命令。

```shell
$ docker container run hello-world
```

结果如下，

```shell
$ docker container run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/


# root @ localhost in ~ [22:06:47]
```



### 进入ubuntu 容器

使用如下方式可以进入ubunut容器，体验ubuntu，

```shell
# root @ localhost in ~ [0:01:52] C:127
$ docker run -it ubuntu /bin/bash
root@82157fef780a:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@82157fef780a:/#

```

前提是你已经通过`docker image pull `把ubuntu镜像拉下来了。

### 启动一个mysql容器

```shell
# root @ localhost in /usr/bin [2:06:20]
$ docker image pull mysql:5.6
5.6: Pulling from library/mysql
7d2977b12acb: Pull complete
5fb8400e7f07: Pull complete
234877fbb165: Pull complete
6fe1021f12f3: Pull complete
7e36fe6b53f0: Pull complete
996ec709c11b: Pull complete
5198b7523387: Pull complete
cc9bdad4dcc0: Pull complete
380cd37ad979: Pull complete
d64465acf034: Pull complete
d4ee6606b3ab: Pull complete
Digest: sha256:2bf1a0a05a6ad437dcac6689e48a9c33774ac92c6213fce2c4196343210592f3
Status: Downloaded newer image for mysql:5.6
docker.io/library/mysql:5.6

# root @ localhost in /usr/bin [2:06:52]
```

可以指定tag拉取mysql镜像。



## Docker镜像的目录存储

### docker拉取的镜像存放在哪儿？

我们知道通过 `docker image pull` 可以拉取镜像，那拉下来的镜像存在哪里呢？

Docker 的镜像是分层存储，每一个镜像都是由很多层，很多个文件组成。

而不同的镜像是共享相同的层的，所以这是一个树形结构，不存在具体哪个文件是 pull 下来的镜像的问题。

不过我们可以在这里看下镜像的repository信息，

```shell
# root @ localhost in /var/lib/docker/image/overlay2 [0:43:27]
$ cat repositories.json |jq
{
  "Repositories": {
    "hello-world": {
      "hello-world:latest": "sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b",
      "hello-world@sha256:d58e752213a51785838f9eed2b7a498ffa1cb3aa7f946dda11af39286c3db9a9": "sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b"
    },
    "ubuntu": {
      "ubuntu:latest": "sha256:74435f89ab7825e19cf8c92c7b5c5ebd73ae2d0a2be16f49b3fb81c9062ab303",
      "ubuntu@sha256:35c4a2c15539c6c1e4e5fa4e554dac323ad0107d8eb5c582d6ff386b383b7dce": "sha256:74435f89ab7825e19cf8c92c7b5c5ebd73ae2d0a2be16f49b3fb81c9062ab303"
    }
  }
}

# root @ localhost in /var/lib/docker/image/overlay2 [0:44:41]
$
```

如果采用的是`overlay2`存储驱动 ,是在`/var/lib/docker/image/overlay2`下面。

docker会在`/var/lib/docker/image`目录下按每个存储驱动的名字创建一个目录，上面例子就是 `overlay2`驱动。

我们 tree看一下目录结构，

```shell
# root @ localhost in /var/lib/docker/image/overlay2 [0:16:30] C:127
$ tree -L 2
.
├── distribution
│   ├── diffid-by-digest
│   └── v2metadata-by-diffid
├── imagedb
│   ├── content
│   └── metadata
├── layerdb
│   ├── mounts
│   ├── sha256
│   └── tmp
└── repositories.json

10 directories, 1 file

# root @ localhost in /var/lib/docker/image/overlay2 [0:16:34]
$
```

这里的关键地方是`imagedb`和`layerdb`目录，看这个目录名字，很明显就是专门用来存储元数据的地方，那为什么区分image和layer呢？因为在docker中，image是由多个layer组合而成的，换句话就是，layer是一个共享的层，可能有多个image会指向同一个layer。

**那如何知道某个image包含哪些layer呢？**

答案就在`imagedb`这个目录中去找。

```shell
# root @ localhost in /var/lib/docker/image/overlay2/imagedb/content/sha256 [1:00:37]
$ ls
2622e6cca7ebbb6e310743abce3fc47335393e79171b9d76ba9d4f446ce7b163
74435f89ab7825e19cf8c92c7b5c5ebd73ae2d0a2be16f49b3fb81c9062ab303
8de95e6026c348c1206d35a9ba7a043ff71885573dace41b14e4236c44cba593
9cfcce23593a93135ca6dbf3ed544d1db9324d4c40b5c0d56958165bfaa2d46a
be0dbf01a0f3f46fc8c88b67696e74e7005c3e16d9071032fa0cd89773771576
bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
# root @ localhost in /var/lib/docker/image/overlay2/imagedb/content/sha256 [1:01:16]
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              74435f89ab78        3 weeks ago         73.9MB
nginx               latest              2622e6cca7eb        4 weeks ago         132MB
mysql               5.6                 8de95e6026c3        4 weeks ago         302MB
mysql               5.7                 9cfcce23593a        4 weeks ago         448MB
mysql               latest              be0dbf01a0f3        4 weeks ago         541MB
hello-world         latest              bf756fb1ae65        6 months ago        13.3kB

# root @ localhost in /var/lib/docker/image/overlay2/imagedb/content/sha256 [1:01:19]
$
```

可以看到`/var/lib/docker/image/overlay2/imagedb/content/sha256`下面5个文件对应的就是5个镜像的ID。

我们以`mysql 5.7`这个镜像为例（9cfcce23593a93135ca6dbf3ed544d1db9324d4c40b5c0d56958165bfaa2d46a）。

```shell
# root @ localhost in /var/lib/docker/image/overlay2/imagedb/content/sha256 [0:24:24]
$ cat 9cfcce23593a93135ca6dbf3ed544d1db9324d4c40b5c0d56958165bfaa2d46a|jq

{
	...
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74",
      "sha256:365386a39e0ea80fcf2a4e3a3cd0e490f073d976133b96dd7f5e2cd1579a8ff5",
      "sha256:c3f46b20a0d3c6532ec63cb2f5475a0a33c8e4c2f22a0a2184d7d79d2f970b37",
      "sha256:66c45123fd43c21cc8be641b73bf2747adf326c6e07d08eadf9b6c829ad575b3",
      "sha256:61cbb8ea64815ee524388e222d32855953ff71bce2a2049185232b3c0463fa93",
      "sha256:44853bb6727490ada4379f3348acbf52b3e7abb63427ce42ca118e11a7b94018",
      "sha256:3a2464d8e0c0697f6fb252a602a6ab95542e5ad10aacc9277d269a182db8dc30",
      "sha256:91ae264962fbfc55b25a1b59378024ef08833c7003823136e73f43985ecda5ee",
      "sha256:8f0182ef7c8cff5ae6b305dff6d7555a249a1e24bfbd94e4f25e75090e763ae3",
      "sha256:ac76579057880b4e115cc46f952aa9b1c92f1f2adbca8ebba810951200e9c288",
      "sha256:c90a34afcab00e4d70d1672b27c4780f6eb881b6cd51c3da492497b15be0b24d"
    ]
  }
}
```

关注rootfs下的diff_ids字段，共有11个元素，其实这11个元素正是对应`mysql`镜像的11个layer，从上往下看，就是底层到顶层，也就是说`13cb14c2acd34e4...`是image的最底层。

既然得到了组成这个image的所有layerID，那么我们就可以带着这些layerID去寻找对应的layer了。

首先看最底层，`13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74`。

我们可以在`/var/lib/docker/image/overlay2/layerdb/sha256/`目录下找到他。

```shell
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256 [23:54:59]
$ ls -ld 13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74
drwx------. 2 root root 71 7月   1 01:24 13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256 [23:55:07]
$ cd 13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74 [23:55:13]
$ ls
cache-id  diff  size  tar-split.json.gz

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74 [23:55:14]
$
```

那是不是说`365386a39e0ea80fcf2a4e3a3cd0e490f073d976133b96dd7f5e2cd1579a8ff5`就是第二层呢？其实不是的！

docker 使用chainID来追踪一个镜像的除最底层的其他layer。

chainID的取值 如下计算：

- 若该镜像层是最底层，那么其chainID 和 diffID （`cat /var/lib/docker/image/overlay2/layerdb/sha256/13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74/diff `）相同。上面也谈到一个镜像的rootfs下面的diff_ids对应这个镜像的不同的layer。比如上面谈的的mysql镜像，最底层是`13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74`。那么我们就可以在`/var/lib/docker/image/overlay2/layerdb/sha256/`找到他。

  ![cap](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/docker_bottomlayer.png)

- 否则，chainID=sha256(父层chainID+" "+本层diffID)

  ```shell
  $ echo -n "sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74 sha256:365386a39e0ea80fcf2a4e3a3cd0e490f073d976133b96dd7f5e2cd1579a8ff5"|sha256sum
  b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3  -
  ```
	比如上面，得到bottom layer的上一层是	`b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3`。
	
	```shell
	# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256 [0:33:51]
	$ ls -ld b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3
	drwx------. 2 root root 85 7月   7 00:59 b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3
	
	# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256 [0:33:54]
	$ cd b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3
	
	# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [0:33:56]
	$ ls
	cache-id  diff  parent  size  tar-split.json.gz
	
	# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [0:33:58]
	$
	```
	
	我们也在`/var/lib/docker/image/overlay2/layerdb/sha256`目录下找到他。

依照上面的规则，我们可以在`/var/lib/docker/image/overlay2/layerdb/sha256`找到某个镜像所有的layer。

使用如下的脚本，我们可以得出这个镜像的所有layerID。

```shell
#!/bin/bash
sha_array=("sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74" "sha256:365386a39e0ea80fcf2a4e3a3cd0e490f073d976133b96dd7f5e2cd1579a8ff5" "sha256:c3f46b20a0d3c6532ec63cb2f5475a0a33c8e4c2f22a0a2184d7d79d2f970b37" "sha256:66c45123fd43c21cc8be641b73bf2747adf326c6e07d08eadf9b6c829ad575b3" "sha256:61cbb8ea64815ee524388e222d32855953ff71bce2a2049185232b3c0463fa93" "sha256:44853bb6727490ada4379f3348acbf52b3e7abb63427ce42ca118e11a7b94018" "sha256:3a2464d8e0c0697f6fb252a602a6ab95542e5ad10aacc9277d269a182db8dc30" "sha256:91ae264962fbfc55b25a1b59378024ef08833c7003823136e73f43985ecda5ee" "sha256:8f0182ef7c8cff5ae6b305dff6d7555a249a1e24bfbd94e4f25e75090e763ae3" "sha256:ac76579057880b4e115cc46f952aa9b1c92f1f2adbca8ebba810951200e9c288" "sha256:c90a34afcab00e4d70d1672b27c4780f6eb881b6cd51c3da492497b15be0b24d")

cnt=${#sha_array[@]}
#echo $cnt

lastlayer=${sha_array[0]}
echo $lastlayer

for ((integer = 1; integer < $cnt; integer++))
do
curlayer=`echo -n "$lastlayer ${sha_array[$integer]}"|sha256sum`
curlayer=`echo $curlayer|sed "s/.$//"`##去除最后一个字符 -
curlayer=`echo $curlayer|sed "s/^[ \s]\{1,\}//g;s/[ \s]\{1,\}$//g"`##去除前后空格
curlayer="sha256:$curlayer"
echo $curlayer
lastlayer=$curlayer
done
```

如下，

```shell
# root @ localhost in ~ [1:20:56]
$ ./test.sh
sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74
sha256:b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3
sha256:57d29ad88aa49f0f439592755722e70710501b366e2be6125c95accc43464844
sha256:12912c9ec523f648130e663d9d4f0a47c1841a0064d4152bcf7b2a97f96326eb
sha256:c37206160543786228aa0cce738e85343173851faa44bb4dc07dc9b7dc4ff1c1
sha256:b704a4a65bf536f82e5d8b86e633d19185e26313de8380162e778feb2852011a
sha256:09687cd9cdf4c704fde969fdba370c2d848bc614689712bef1a31d0d581f2007
sha256:ab8bf065b402b99aec4f12c648535ef1b8dc954b4e1773bdffa10ae2027d3e00
sha256:c04c087c2af9abd64ba32fe89d65e6d83da514758923de5da154541cc01a3a1e
sha256:17e8b88858e400f8c5e10e7cb3fbab9477f6d8aacba03b8167d34a91dbe4d8c1
sha256:98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be

# root @ localhost in ~ [1:20:59]
$

```

不过需要注意的是，我们上面说的都是镜像以及镜像对应的layer的元数据。那每一层的layer的数据是存放在哪里的呢？

### layerdb，镜像layer的元数据

关注`/var/lib/docker/image/overlay2/layerdb/sha256`下面的目录。

```bash
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256 [1:47:14] C:2
$ ls
059442698ef65fe8545e4fe9657988a10329b9c3663b368ae7ee0007a9c43949
086d66e8d1cb0d52e9337eabb11fb9b95960e2e1628d90100c62ea5e8bf72306
09687cd9cdf4c704fde969fdba370c2d848bc614689712bef1a31d0d581f2007
12912c9ec523f648130e663d9d4f0a47c1841a0064d4152bcf7b2a97f96326eb
13389dfab8e64fcaafadde604432773a493c89d7c6ad6c9075547db0c7e9e31b
13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74
1644f130455c90b0fe2b32b80ffa2d285cacdb21e13b717b370d4df0e63e6778
17e8b88858e400f8c5e10e7cb3fbab9477f6d8aacba03b8167d34a91dbe4d8c1
...
f37c61ee1973b18c285d0d5fcf02da4bcdb1f3920981499d2a20b2858500a110
```

前面说过，`/var/lib/docker/image/overlay2/layerdb/sha256`下面存放的是镜像的各层layer的元数据，我们看下有哪些。

还是以`mysql 5.7`镜像为例，以倒数第二层`b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3`为例。

```bash
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:49:51]
$ ls
cache-id  diff  parent  size  tar-split.json.gz

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:49:52]
$ cat diff
sha256:365386a39e0ea80fcf2a4e3a3cd0e490f073d976133b96dd7f5e2cd1579a8ff5#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:49:55]
$ cat parent
sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:49:58]
$ cat size
328574#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:50:01]
$ cat cache-id
4d3692d2d7417047be380de1a9fabab3c1a9ccebae94e72bb861ebfbc2e3d8a1#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/sha256/b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3 [1:50:09]
$
```



- diff ，表示该层的diff id。我们上面谈过，假如最底层的layer为1的话，那么第n+1层的chainID是通过第n层的chainID和第n+1层的diff id计算的到的。

- parent，表示该层的父层的chainID，可以看到`b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3`的父层chainID为`13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74`，符合预期

- size，表示该层的大小，单位为字节

- cache-id，内容为一个uuid，指向当前layer本地的真正存储位置，那么Layer本地真正的存储位置又在何处呢？便是上面提到的`/var/lib/docker/overlay2`目录下：

  ```
  # root @ localhost in /var/lib/docker/overlay2/4d3692d2d7417047be380de1a9fabab3c1a9ccebae94e72bb861ebfbc2e3d8a1 [1:56:26]
  $ ls
  committed  diff  link  lower  work
  
  # root @ localhost in /var/lib/docker/overlay2/4d3692d2d7417047be380de1a9fabab3c1a9ccebae94e72bb861ebfbc2e3d8a1 [1:56:27]
  $
  
  ```

  

### image/overlay2/layerdb/mounts

前面谈到了容器的镜像的分层，主要是讲了只读的镜像层。

对于容器来说，最上面还有1个读写层？这个读写层（容器层）的元数据在哪里呢？

就在`/var/lib/docker/image/overlay2/layerdb/mounts`。

```shell
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts [0:35:13]
$ ls
08e7bc15643a6007b5d5a83924609098a17bce6fa5e762692d5c3044441b3555
1a16698a9171bc346a1ab3f38af216e717961e37fef43a27d0bdff5b01312862
22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2
39e2b86aafb681cb108ddba0d2bca4df78e4c6a076fce3645041e474ec8038c1
421b4dda3fb02cf3501f455da361689b92acdd2f501d7a66b999d679c4330d37
4da4d33017bccec9f652b1367889e05ee69e3819377590b5a0ad0790d9ca8f2c
51f328e0ef59d81656d96d2956edd79e4ffbd75c180abe8caa3942444f77e254
60cb48e437a899c9098dd1f01c3c8bb706d83fd04cfbc6caee99f1d1e6688fbb
73514c85ac1c77c18ff76753b9cf76b92a7aef57ede9d33c01e0c8f816347061
76094f299ac09e1c118046ab546af159735fe845ce434c44c260f5697faf6e39
7802c0ee9df78328ba0763a772ce526462634bcf654795ba073c1b1e06b4cc51
82157fef780af00830d161c3aff0d119b3ce2c3af72f671f737f30cedde787b4
97d1eaba61126712182a0ee1de4fbf5eec949efbe3ca15df2f6515a68888a95c
99c34998fe083bd9e64d91b2116abdb7d0fdcb6aa0c176de624903f38e64610f
c05163f474ee057edfb03e46226f68a25408add072f0b93c675851d09bfe57c4
f7a6a316d34bcd72f31db72c89f873f130ce946091422607e227c3d8df276c07
fc992d035ff63c391f27ca0594097227d2e7f7ef47911377548f1fff63c2c771

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts [0:35:13]
$
```

可以看到，有17个目录，这17个对应的就是17个容器的ID。

```bash
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts [0:35:13]
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
22b11143a9b8        mysql:5.7           "docker-entrypoint.s…"   2 weeks ago         Exited (0) 34 hours ago                       mysql-5.7
97d1eaba6112        mysql:5.6           "docker-entrypoint.s…"   2 weeks ago         Exited (0) 34 hours ago                       mysql-test
421b4dda3fb0        mysql               "docker-entrypoint.s…"   3 weeks ago         Exited (0) 3 weeks ago                        loving_lichterman
08e7bc15643a        mysql               "docker-entrypoint.s…"   3 weeks ago         Exited (1) 3 weeks ago                        pedantic_brown
7802c0ee9df7        mysql               "docker-entrypoint.s…"   3 weeks ago         Exited (1) 3 weeks ago                        busy_robinson
76094f299ac0        nginx               "/docker-entrypoint.…"   3 weeks ago         Exited (0) 3 weeks ago                        serene_ellis
fc992d035ff6        nginx               "/docker-entrypoint.…"   3 weeks ago         Exited (0) 3 weeks ago                        condescending_wilbur
82157fef780a        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        admiring_davinci
73514c85ac1c        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        dazzling_banzai
51f328e0ef59        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        awesome_ride
4da4d33017bc        ubuntu              "cat /etc/os-release"    4 weeks ago         Exited (0) 3 weeks ago                        jolly_proskuriakova
c05163f474ee        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        trusting_spence
1a16698a9171        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        sweet_varahamihira
f7a6a316d34b        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        frosty_hamilton
99c34998fe08        ubuntu              "/bin/bash"              4 weeks ago         Exited (0) 3 weeks ago                        eloquent_chatelet
39e2b86aafb6        hello-world         "/hello"                 4 weeks ago         Exited (0) 4 weeks ago                        priceless_goldstine
60cb48e437a8        hello-world         "/hello"                 4 weeks ago         Exited (0) 4 weeks ago                        frosty_payne

```

还是以`mysql 5.7`容器为例，即`22b11143a9b8`这个。

```bash
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts/22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2 [0:43:16]
$ ls
init-id  mount-id  parent

# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts/22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2 [0:43:28]
$ cat init-id
885c42c9d4f8a3b91a7ac832a11d65839503fad8b01aa972a470e91bf7c121e9-init#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts/22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2 [0:43:30]
$ cat mount-id
885c42c9d4f8a3b91a7ac832a11d65839503fad8b01aa972a470e91bf7c121e9#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts/22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2 [0:43:31]
$ cat parent
sha256:98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be#
# root @ localhost in /var/lib/docker/image/overlay2/layerdb/mounts/22b11143a9b8460d4e4e038abd96d8ecea2ed3b386ad4db126bd1c0498f7cce2 [0:43:35]
$

```

有三个文件

- init-id，
- mount-id，容器读写层的mountID
- parent，表示这个容器层的上一层是谁，上面例子中，parent是`98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be`，这个是啥？`98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be`是镜像`mysql 5.7`的最顶层，就是说容器层的parent就是镜像的最顶层

可以看到对于每个运行过的容器来说，除了和镜像有关的layer的元数据是放在`/var/lib/docker/image/overlay2/layerdb/sha256`下面。和当前这个运行过的容器来说，还有2个layer需要关注，分别是只读的init-layer和可写的mount layer。

init-layer包含了docker为容器准备的一些文件，mount-layer则用于保存以后对rootfs的增删改操作结果。

docker将容器的layer和image的layer的元数据放在不同的目录。

一个是

```
/var/lib/docker/image/overlay2/layerdb/mounts
```

一个是

```
/var/lib/docker/image/overlay2/layerdb/sha256
```



### layer的数据

上面我们讲了容器的元数据。

容器的元数据主要包括镜像的元数据，以及docker为这个容器创建的2个新layer的元数据。

镜像的元数据放在`/var/lib/docker/image/overlay2/layerdb/sha256`，新建的2层layer的元数据放在`/var/lib/docker/image/overlay2/layerdb/mounts`。

那layer的数据放在那里呢？

```
/var/lib/docker/overlay2
```

这个目录存放的是各个layer的数据。

我们还是以`mysql 5.7`容器为例，我们看下docker为这个容器新建的2层layer都有些啥数据。

```bash
$ ls  /var/lib/docker/overlay2/885c42c9d4f8a3b91a7ac832a11d65839503fad8b01aa972a470e91bf7c121e9
885c42c9d4f8a3b91a7ac832a11d65839503fad8b01aa972a470e91bf7c121e9/
885c42c9d4f8a3b91a7ac832a11d65839503fad8b01aa972a470e91bf7c121e9-init/
```

init-layer和mount-layer。


### docker镜像生成的container存在哪儿？

默认的话，

```shell
# root @ localhost in /var/lib/docker/containers [0:48:14]
$ ls
1a16698a9171bc346a1ab3f38af216e717961e37fef43a27d0bdff5b01312862
39e2b86aafb681cb108ddba0d2bca4df78e4c6a076fce3645041e474ec8038c1
4da4d33017bccec9f652b1367889e05ee69e3819377590b5a0ad0790d9ca8f2c
51f328e0ef59d81656d96d2956edd79e4ffbd75c180abe8caa3942444f77e254
60cb48e437a899c9098dd1f01c3c8bb706d83fd04cfbc6caee99f1d1e6688fbb
73514c85ac1c77c18ff76753b9cf76b92a7aef57ede9d33c01e0c8f816347061
82157fef780af00830d161c3aff0d119b3ce2c3af72f671f737f30cedde787b4
99c34998fe083bd9e64d91b2116abdb7d0fdcb6aa0c176de624903f38e64610f
c05163f474ee057edfb03e46226f68a25408add072f0b93c675851d09bfe57c4
f7a6a316d34bcd72f31db72c89f873f130ce946091422607e227c3d8df276c07

# root @ localhost in /var/lib/docker/containers [0:48:15]
$
```

可以看到有10个container，那一行行的长串字符就是容器ID。

```shell
# root @ localhost in /var/lib/docker/containers [0:49:55]
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS                        PORTS               NAMES
82157fef780a        ubuntu              "/bin/bash"             48 minutes ago      Exited (0) 16 minutes ago                         admiring_davinci
73514c85ac1c        ubuntu              "/bin/bash"             48 minutes ago      Exited (127) 48 minutes ago                       dazzling_banzai
51f328e0ef59        ubuntu              "/bin/bash"             23 hours ago        Exited (0) 52 minutes ago                         awesome_ride
4da4d33017bc        ubuntu              "cat /etc/os-release"   23 hours ago        Exited (0) 23 hours ago                           jolly_proskuriakova
c05163f474ee        ubuntu              "/bin/bash"             23 hours ago        Exited (0) 23 hours ago                           trusting_spence
1a16698a9171        ubuntu              "/bin/bash"             36 hours ago        Exited (0) 23 hours ago                           sweet_varahamihira
f7a6a316d34b        ubuntu              "/bin/bash"             36 hours ago        Exited (0) 36 hours ago                           frosty_hamilton
99c34998fe08        ubuntu              "/bin/bash"             2 days ago          Exited (0) 2 days ago                             eloquent_chatelet
39e2b86aafb6        hello-world         "/hello"                2 days ago          Exited (0) 2 days ago                             priceless_goldstine
60cb48e437a8        hello-world         "/hello"                2 days ago          Exited (0) 2 days ago                             frosty_payne

# root @ localhost in /var/lib/docker/containers [0:49:56]
$
```

## docker常用命令

1. `docker info`
   Display system-wide information.

2. `docker version`
   Show the Docker version information.

3. `docker image ls`
   列出本机的所有 image 文件。

4. `docker image pull library/hello-world`，拉取镜像到本地。`docker image pull`和`docker pull`是一回事。
   由于 Docker 官方提供的 image 文件，都放在[`library`](https://hub.docker.com/r/library/)组里面，所以它是默认组，可以省略。因此，上面的命令可以写成下面这样。

   ```shell
   docker image pull hello-world
   ```

5. `docker container run`，会从image文件，生成一个正在运行的容器实例，`docker container run`具有自动抓取image文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。`docker container run `和`docker run`是一回事？**对，完全相同！**

6. `docker container kill [containID]`，有些容器不会自动终止，因为提供的是服务。对于那些不会自动终止的容器，必须使用`docker container kill [containID]`手动终止。
   如果你安装了一些软件，比如 git，然后 exit退出，再次执行 run 命令进入，你会发现 git 找不到了，这是因为每次执行 run 命令都将从我们下载的镜像新建一个容器，而 git 是装在上一个容器里，自然找不到了。

7. `docker container run -it ubuntu bash`，这里主要关注 `-it`选项。`-it`参数可以将当前终端连接到容器的Shell终端之上。-i 表示interactive交互式，-t 表示得到一个 terminal。

8. `docker container ls -a`，查看所有容器。不加`-a`，只能看到正在运行的容器。

9. `docker ps -a`，效果和`docker container ls -a`一样。我们使用`docker ps -a`能找到之前通过`exit`退出的容器，比如

   ```shell
   # root @ localhost in ~ [1:55:24]
   $ docker container ls -a
   CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS                      PORTS               NAMES
   4da4d33017bc        ubuntu              "cat /etc/os-release"   9 minutes ago       Exited (0) 9 minutes ago                        jolly_proskuriakova
   c05163f474ee        ubuntu              "/bin/bash"             9 minutes ago       Exited (0) 9 minutes ago                        trusting_spence
   1a16698a9171        ubuntu              "/bin/bash"             13 hours ago        Exited (0) 14 minutes ago                       sweet_varahamihira
   f7a6a316d34b        ubuntu              "/bin/bash"             13 hours ago        Exited (0) 13 hours ago                         frosty_hamilton
   99c34998fe08        ubuntu              "/bin/bash"             28 hours ago        Exited (0) 28 hours ago                         eloquent_chatelet
   39e2b86aafb6        hello-world         "/hello"                28 hours ago        Exited (0) 28 hours ago                         priceless_goldstine
   60cb48e437a8        hello-world         "/hello"                28 hours ago        Exited (0) 28 hours ago                         frosty_payne
   
   # root @ localhost in ~ [1:56:18]
   $
   ```

   其实上面展示的这些容器都是之前通过`docker container run`生成的的container，只不过当前没有在运行。怎么进入之前创建的容器呢？`docker start -i`即可。

10. `docker start -i xxxxxxxx`，启动一个容器，其中`xxxxxxxx`表示container id。container id 可以通过`docker ps -a`来查询。

11. `docker search xxx`，在仓库里（比如前面谈到的国内源，或者官方源）搜索某个镜像。

    ```shell
    # root @ localhost in ~ [23:51:01] C:1
    $ docker search ubuntu --limit 1
    NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
    ubuntu              Ubuntu is a Debian-based Linux operating sys…   11054               [OK]
    
    # root @ localhost in ~ [23:51:11]
    $
    ```

    搜索到这个镜像，我们就可以使用`docker image pull ubuntu`把这个镜像拉下来，然后使用`docker container run -it ubuntu bash`生成一个运行的容器实例，并进入他的bash。
    
12. 优雅地停止一个运行的容器，`docker stop ContainerId`，`docker stop -t=60`，`-t`参数，关闭容器的限时，如果超时未能关闭则用`docker kill`强制关闭，默认值10s，这个时间用于容器的自己保存状态。

13. 直接关闭容器，`docker kill ContainerId`。与`docker stop`的区别，参见这里，https://www.jb51.net/article/96617.htm。

14. 重启容器服务，`docker restart ContainerId`，也可以加`-t`参数，含义和`docker stop -t `中的`-t`一样。

15. `docker run -i -t -d`，`-i, -t, -d`参数，这些表示啥呢？

    - `-d`，表示在后台运行。在不指定`-d`的情况下，容器默认是前台运行的，可以看到容器运行时候的输入输出以及错误信息日志。
    - `-i`，Keep STDIN open even if not attached。
    - xxxx
    - xxxx
    - xxxx
    - xxx

16. `docker exec -it ${CONTAINER NAME/ID} /bin/bash`，进入容器内

17. `docker run IMAGE[:TAG]`，run某个版本的镜像。比如`docker run -itd --name mysql-test -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.6`，启动的就是mysql 5.6。

18. 如果你觉得容器的名字不好记，可以rename一下。`docker rename [CONTAINER ID] newname`。

19. 拷贝文件，`docker container cp`，用于从正在运行的 Docker 容器里面，将文件拷贝到本机；或者将本机的文件拷贝到容器。

    ```
    docker container cp [containID]:[/path/to/file] .
    ```

20.     删除容器，`docker rm xxx`。

## 参考

1. [阮一峰的Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

