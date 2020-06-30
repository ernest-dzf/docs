# docker存储驱动

### **镜像的分层特性**

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

上面的示例是从docker官方镜像仓库拉取一个nginx:latest镜像，可以看到在拉取镜像时，是一层一层的拉取的。

事实上镜像也是这么一层一层地存储在磁盘上的。通常一个应用镜像包含多层，如下：

