# 利用 rsync 和 inotify 同步两个机器的文件#
我们有时候会有这个需求，需要将A机器某个目录下的文件实时同步到B机器某个目录下的文件。可以利用rsync和inotify来解决。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/rsync_inotify.png)