# docker存储驱动

## 镜像的分层特性

我们知道docker的启动是依赖于image，docker在启动之前，需要先拉取image，然后启动。

多个容器可以使用同一个image启动。那么问题来了：这些容器是共用一个image，还是各自将这个image复制了一份，然后各自独立运行呢？

我们假设每个容器都复制了一份这个image，然后各自独立运行，那么就意味着，启动多少个容器，就需要复制多少个image，毫无疑问这是对空间的一种巨大浪费。

事实上，在容器的设计当中，通过同一个Image启动的容器，全部都共享这个image，而并不复制。

那么问题又随之而来：既然所有的容器都共用这一个image，那么岂不是我在任意一个容器中所做的修改，在其他容器中都可见？如果我一个容器要将一个配置文件修改成A，而另一个容器同样要将这个文件修改成B，两个容器岂不是会产生冲突？

我们把上面的问题放一放，先来看下面一个拉取镜像的示例：

```shell
# root @ localhost in /var/lib/docker/overlay2 [1:24:05]
$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
8559a31e96f4: Pull complete
8d69e59170f7: Pull complete
3f9f1ec1d262: Pull complete
d1f5ff4f210d: Pull complete
1e22bfa8652e: Pull complete
Digest: sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

# root @ localhost in /var/lib/docker/overlay2 [1:24:28]
$

```

上面的示例是从docker官方镜像仓库拉取一个nginx:latest镜像，可以看到在拉取镜像时，是一层一层地拉取的。

事实上镜像也是这么一层一层地存储在磁盘上的。通常一个应用镜像包含多层，如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/docker_layer.png)

我们首先需要明确一点，镜像是只读的。每一层都只读。

在上图上，我们可以看到，在内核之上，最底层首先是一个基础镜像层，这里是一个ubuntu的基础镜像。

因为镜像的只读特性，如果我们想要在这个ubuntu的基础镜像上安装一个emacs编辑器，则只能在基础镜像之上，再构建一层新的镜像层。

同样的道理，如果想要在当前的emacs镜像层之上添加一个apache，则只能在其上再构建一个新的镜像层。

而这即是镜像的分层特性。

## 容器读写层的工作原理

我们刚刚在说镜像的分层特性的时候说到镜像是只读的。

而事实上当我们使用镜像启动一个容器的时候，我们其实是可以在容器里随意读写的，从结果上看，似乎与镜像的只读特性相悖。

我们继续看上面的图，其实可以看到在镜像的最上层，还有一个**读写层**。而这个读写层，即在容器启动时为当前容器单独挂载。

每一个容器在运行时，都会基于当前镜像在其最上层挂载一个**读写层**。而用户针对容器的所有操作都在读写层中完成。一旦容器销毁，这个读写层也随之销毁。

> 容器=镜像+读写层

而我们针对这个读写层的操作，主要基于两种方式：**写时复制**和**用时分配**。

## 写时复制

所有驱动都用到的技术——写时复制（CoW）。

CoW就是copy-on-write，表示只在需要写时才去复制，这个是针对已有文件的修改场景。

比如基于一个image启动多个Container，如果为每个Container都去分配一个image一样的文件系统，那么将会占用大量的磁盘空间。

而CoW技术可以让所有的容器共享image的文件系统，所有数据都从image中读取，只有当要对文件进行写操作时，才从image里把要写的文件复制到自己的文件系统进行修改。

所以无论有多少个容器共享同一个image，所做的写操作都是对从image中复制到自己的文件系统中的复本上进行，并不会修改image的源文件，且多个容器操作同一个文件，会在每个容器的文件系统里生成一个复本，每个容器修改的都是自己的复本，相互隔离，相互不影响。使用CoW可以有效地提高磁盘的利用率。

## 用时分配

用时分配是用在原本没有这个文件的场景，只有在要新写入一个文件时才分配空间，这样可以提高存储资源的利用率。

比如启动一个容器，并不会为这个容器预分配一些磁盘空间，而是当有新文件写入时，才按需分配新空间。

## Docker存储驱动

接下来我们说一说，这些分层的镜像是如何在磁盘中存储的。

docker提供了多种存储驱动来实现不同的方式存储镜像，下面是常用的几种存储驱动：

- AUFS
- OverlayFS
- Devicemapper
- Btrfs
- ZFS

可以通过`docker info`查看当前所使用的存储 驱动，

```shell
# root @ localhost in /var/lib/docker/overlay2 [2:13:52]
$ docker info
Client:
 Debug Mode: false

Server:
 Containers: 10
  Running: 0
  Paused: 0
  Stopped: 10
 Images: 3
 Server Version: 19.03.12
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  ...
```

可以看到使用的是 overlay2。

### AUFS

AUFS（AnotherUnionFS）是一种Union FS，是文件级的存储驱动。

AUFS是一个能透明覆盖一个或多个现有文件系统的层状文件系统，把多层合并成文件系统的单层表示。

简单来说就是支持将不同目录挂载到同一个虚拟文件系统下的文件系统。

这种文件系统可以一层一层地叠加修改文件。无论底下有多少层都是只读的，只有最上层的文件系统是可写的。

当需要修改一个文件时，AUFS创建该文件的一个副本，使用CoW将文件从只读层复制到可写层进行修改，结果也保存在可写层。

在Docker中，底下的只读层就是image，可写层就是Container。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/docker_aufs.png)

### OverlayFS

Overlay是Linux内核3.18后支持的，也是一种Union FS，**和AUFS的多层不同的是Overlay只有两层**：一个upper文件系统和一个lower文件系统，分别代表Docker的容器层和镜像层。

当需要修改一个文件时，使用CoW将文件从只读的lower复制到可写的upper进行修改，结果也保存在upper层。

在Docker中，底下的只读层就是image，可写层就是Container。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/docker_overlayFS.jpg)

我们根据镜像，使用`docker run xxxx`命令，运行一个容器之后，会有对应的containerID。默认是会在`/var/lib/docker/containers`目录下，

```shell
# root @ localhost in /var/lib/docker/containers [1:12:22]
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

# root @ localhost in /var/lib/docker/containers [1:12:23]
$
```

可以看到，这里有10个container。你使用`docker run xxx`，运行一个容器之后，这个容器就创建了。后面你可以通过`docker start -i containerID`来再次运行这个容器。

每个容器有自己的读写层，你在容器里面创建修改的文件，都会保存在每个容器各自的读写层里。

如果使用的是`overlay2`驱动，那么就是保存在`/var/lib/docker/overlay2`目录下。



6e048e2f4d5aa3098ae07530a1da0ae9ec591f662fa443da13d59b20bed64b72-init



51f328e0ef59



## 参考文章

1. [一步步了解 Docker 存储驱动](https://zhuanlan.zhihu.com/p/73147396)

2.  [Docker镜像存储-overlayfs](https://www.cnblogs.com/wdliu/p/10483252.html)

