# 安装

1. 官网下载`percona-toolkit-3.2.1_x86_64.tar.gz`，地址是，`https://www.percona.com/downloads/percona-toolkit/LATEST/`

2. 解压进入目录，查看`INSTALL`安装说明

3. 安装步骤如下，

   ```
   Quick Install
   -------------
   
      perl Makefile.PL
      make
      make test
      make install
   
   Detailed Install
   ----------------
   ```

   

4. 第三步可能由于缺失一些包，导致报错。常见的，你需要安装`yum -y install perl-CPAN`，`yum install perl-DBD-MySQL`，`yum -y install perl-Digest-MD5`

5. 安装完成之后，预期你可以看到如下，

   ```
   [root@VM_0_15_centos percona-toolkit-3.2.1]# pt
   pt-align                  pt-fk-error-logger        pt-pg-summary             pt-summary
   pt-archiver               pt-heartbeat              pt-pmp                    pt-table-checksum
   ptaskset                  pt-index-usage            pt-query-digest           pt-table-sync
   pt-config-diff            pt-ioprofile              pt-secure-collect         pt-table-usage
   pt-deadlock-logger        pt-kill                   pt-show-grants            pt-upgrade
   pt-diskstats              pt-mext                   pt-sift                   pt-variable-advisor
   pt-duplicate-key-checker  pt-mongodb-query-digest   pt-slave-delay            pt-visual-explain
   pt-fifo-split             pt-mongodb-summary        pt-slave-find             ptx
   pt-find                   pt-mysql-summary          pt-slave-restart          
   pt-fingerprint            pt-online-schema-change   pt-stalk                  
   [root@VM_0_15_centos percona-toolkit-3.2.1]# pt
   
   ```

   

# 使用

`pt-online-schema-change`参数很多，我们只需要了解常用的几个就行。

- `--user=`，连接mysql的用户名
- `--password=`，连接mysql的密码
- `--host=`，连接mysql的地址
- `P`，连接mysql的端口号，注意这里参数和上面不一样。没有以`--`开头
- `D`，连接mysql的database
- `t`，Table to alter，需要alter的表
- `--alter`，修改表结构的语句
- `--execute`，执行修改表结构
- `--charset=`，使用什么编码
- `--no-version-check`，不检查版本



示例语句：

```shell
pt-online-schema-change --charset=utf8 --no-version-check --user=victor --password=199233 --host=172.19.0.15 P=3306,D=victor,t=sbtest1 --alter="ADD COLUMN column1 tinyint(4) DEFAULT NULL" --execute
```

