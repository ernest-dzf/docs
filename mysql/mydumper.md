# mydumper

## 安装

1. 安装依赖包

   ```shell
   yum groupinstall "Development Libraries"
   yum install glib2-devel mysql-devel zlib-devel pcre-devel openssl-devel cmake make
   ```

2. 下载二进制包

   ```shell
   wget https://launchpadlibrarian.net/225370879/mydumper-0.9.1.tar.gz
   ```

3. install

   ```shell
   tar zxvf mydumper-0.9.1.tar.gz
   cd mydumper-0.9.1/
   cmake .
   make && make install
   ```

安装完成后，有两个二进制文件，在`/usr/local/bin`目录下。

```shell
[root@VM_0_15_centos 3306]# cd /usr/local/bin/
[root@VM_0_15_centos bin]# ls
mydumper  myloader
[root@VM_0_15_centos bin]# 

```



## 使用

## 分析