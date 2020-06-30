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







## Docker镜像的目录存储

### docker拉取的镜像存放在哪儿？

我们知道通过 `docker image pull` 可以拉取镜像，那拉下来的镜像存在哪里呢？

默认的话，

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

是在`/var/lib/docker/image`下面。

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

## 参考

1. [阮一峰的Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

