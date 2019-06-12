[toc]
# nginx 学习笔记 #

## 说明 ##

后面会有一些测试代码，我们是通过一台本地mac和腾讯云上的云主机进行测试的。

nginx安装在腾讯云的云主机上。

在本机mac上配置了host，如下：

	# victor @ VICTORDONG-MB0 in ~ [1:39:10] C:64
	$ ping hkcvm -c 1
	PING hkcvm (124.156.182.201): 56 data bytes
	64 bytes from 124.156.182.201: icmp_seq=0 ttl=50 time=63.545 ms
	
	--- hkcvm ping statistics ---
	1 packets transmitted, 1 packets received, 0.0% packet loss
	round-trip min/avg/max/stddev = 63.545/63.545/63.545/0.000 ms
	
	# victor @ VICTORDONG-MB0 in ~ [1:39:13] 
	$ ping wind.com -c 1
	PING wind.com (124.156.182.201): 56 data bytes
	64 bytes from 124.156.182.201: icmp_seq=0 ttl=50 time=61.300 ms
	
	--- wind.com ping statistics ---
	1 packets transmitted, 1 packets received, 0.0% packet loss
	round-trip min/avg/max/stddev = 61.300/61.300/61.300/0.000 ms
	
	# victor @ VICTORDONG-MB0 in ~ [1:39:28] 
	$ 

开始的时候，使用了`150.109.76.79`这个公网ip，后面ip被封了，就换了个ip，就是124.156.182.201。

## 安装 ##
步骤如下：

1. 下载源码，`wget http://nginx.org/download/nginx-1.16.0.tar.gz`
2. 解压到目录，`tar -zxvf nginx-1.16.0.tar.gz`，如下：

		[root@VM_0_15_centos nginx-1.16.0]# pwd
		/root/src/nginx-1.16.0
		[root@VM_0_15_centos nginx-1.16.0]# ls
		auto     CHANGES.ru  configure  html     Makefile  objs    src
		CHANGES  conf        contrib    LICENSE  man       README
		[root@VM_0_15_centos nginx-1.16.0]# 
3. `./configure --prefix=/usr/local/nginx`，指定安装目录，不清楚的可以`./configure --help`查看一下具体configure怎么用。建议事先将目录`/usr/local/nginx`建立起来，然后再configure。
4. configure之后会在当前目录生成`Makefile`文件，然后直接`make && make install`。安装完毕。
5. 可以在`/usr/local/nginx`目录看到安装好的nginx。

		[root@VM_0_15_centos nginx]# ls
		client_body_temp  fastcgi_temp  logs        sbin       uwsgi_temp
		conf              html          proxy_temp  scgi_temp
		[root@VM_0_15_centos nginx]# pwd
		/usr/local/nginx
		[root@VM_0_15_centos nginx]# 

6. 加入service 启动脚本，放入`/etc/init.d/`目录下。


	需要注意下面脚本中需要更改的几个地方。
	
	`# pidfile:     /var/run/nginx.pid`, 需要更改nginx的配置文件`nginx.conf`，将pid路径改为`/var/run/nginx.pid`。
	
		#!/bin/sh
		#
		# nginx - this script starts and stops the nginx daemon
		#
		# chkconfig:   - 85 15
		# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
		#               proxy and IMAP/POP3 proxy server
		# processname: nginx
		# config:      /etc/nginx/nginx.conf
		# config:      /etc/sysconfig/nginx
		# pidfile:     /var/run/nginx.pid
		
		# Source function library.
		. /etc/rc.d/init.d/functions
		
		# Source networking configuration.
		. /etc/sysconfig/network
		
		# Check that networking is up.
		[ "$NETWORKING" = "no" ] && exit 0
		
		nginx="/usr/local/nginx/sbin/nginx"
		prog=$(basename $nginx)
		
		NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
		
		[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
		
		lockfile=/var/lock/subsys/nginx
		
		make_dirs() {
		   # make required directories
		   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
		   if [ -n "$user" ]; then
		      if [ -z "`grep $user /etc/passwd`" ]; then
		         useradd -M -s /bin/nologin $user
		      fi
		      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
		      for opt in $options; do
		          if [ `echo $opt | grep '.*-temp-path'` ]; then
		              value=`echo $opt | cut -d "=" -f 2`
		              if [ ! -d "$value" ]; then
		                  # echo "creating" $value
		                  mkdir -p $value && chown -R $user $value
		              fi
		          fi
		       done
		    fi
		}
		
		start() {
		    [ -x $nginx ] || exit 5
		    [ -f $NGINX_CONF_FILE ] || exit 6
		    make_dirs
		    echo -n $"Starting $prog: "
		    daemon $nginx -c $NGINX_CONF_FILE
		    retval=$?
		    echo
		    [ $retval -eq 0 ] && touch $lockfile
		    return $retval
		}
		
		stop() {
		    echo -n $"Stopping $prog: "
		    killproc $prog -QUIT
		    retval=$?
		    echo
		    [ $retval -eq 0 ] && rm -f $lockfile
		    return $retval
		}
		
		restart() {
		    configtest || return $?
		    stop
		    sleep 1
		    start
		}
		
		reload() {
		    configtest || return $?
		    echo -n $"Reloading $prog: "
		    killproc $nginx -HUP
		    RETVAL=$?
		    echo
		}
		
		force_reload() {
		    restart
		}
		
		configtest() {
		  $nginx -t -c $NGINX_CONF_FILE
		}
		
		rh_status() {
		    status $prog
		}
		
		rh_status_q() {
		    rh_status >/dev/null 2>&1
		}
		
		case "$1" in
		    start)
		        rh_status_q && exit 0
		        $1
		        ;;
		    stop)
		        rh_status_q || exit 0
		        $1
		        ;;
		    restart|configtest)
		        $1
		        ;;
		    reload)
		        rh_status_q || exit 7
		        $1
		        ;;
		    force-reload)
		        force_reload
		        ;;
		    status)
		        rh_status
		        ;;
		    condrestart|try-restart)
		        rh_status_q || exit 0
		            ;;
		    *)
		        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
		        exit 2
		esac

7. 现在可以通过`service start nginx`来启动nginx了。

		[root@VM_0_15_centos init.d]# service nginx start
		Starting nginx (via systemctl):                            [  OK  ]
		[root@VM_0_15_centos init.d]# 
		
8. 如果想要开机启动可以使用`chkconfig`。`chkconfig --add nginx`，`chkconfig nginx --level 4 on`。

		[root@VM_0_15_centos init.d]# chkconfig --list
		
		注：该输出结果只显示 SysV 服务，并不包含
		原生 systemd 服务。SysV 配置数据
		可能被原生 systemd 配置覆盖。 
		
		      要列出 systemd 服务，请执行 'systemctl list-unit-files'。
		      查看在具体 target 启用的服务请执行
		      'systemctl list-dependencies [target]'。
		
		mysql.server    0:关    1:关    2:开    3:开    4:开    5:开    6:关
		netconsole      0:关    1:关    2:关    3:关    4:关    5:关    6:关
		network         0:关    1:关    2:开    3:开    4:开    5:开    6:关
		nginx           0:关    1:关    2:开    3:开    4:开    5:开    6:关
		[root@VM_0_15_centos init.d]# 
9. 关于service 脚本可以参考[这里](https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/)。

## 配置文件

nginx配置文件名称为`nginx.conf`，默认的话会是在`/usr/local/nginx/conf/nginx.conf`。

	[root@VM_0_15_centos conf]# pwd
	/usr/local/nginx/conf
	[root@VM_0_15_centos conf]# ls -l nginx.conf
	-rw-r--r-- 1 nginx nginx 2687 5月  26 18:58 nginx.conf
	[root@VM_0_15_centos conf]# 
	
### 配置文件结构

1. 全局配置
2. events（网络连接相关）
3. http

如下：


	worker_processes  1;
	
	pid        /var/run/nginx.pid;
	
	events {
	    worker_connections  1024;
	}
	
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	
	
	
	    sendfile        on;
	
	    keepalive_timeout  65;
	
	
	    server {
	        listen       80;
	        server_name  localhost;

	
	        location / {
	            root   html;
	            index  index.html index.htm;
	        }
	
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }

	    }
	
	
	    # another virtual host using mix of IP-, name-, and port-based configuration
	    #
	    #server {
	    #    listen       8000;
	    #    listen       somename:8080;
	    #    server_name  somename  alias  another.alias;
	
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #}
	
		}


可以看到 `http` 里面包含`server`，`server`里面包含`location`。



## 虚拟主机

nginx可以配置虚拟主机，就是在`nginx.conf`配置文件中写配置。

最佳实践是在nginx的目录新建一个`vhost`目录，然后每个虚拟主机的配置都放在`vhost`目录下面。然后再在nginx配置文件`nginx.conf`中include这些配置。

	[root@VM_0_15_centos vhost]# ls
	victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# pwd
	/usr/local/nginx/conf/vhost
	[root@VM_0_15_centos vhost]# cd ../
	[root@VM_0_15_centos conf]# ls -l nginx.conf
	-rw-r--r-- 1 nginx nginx 377 5月  26 22:23 nginx.conf
	[root@VM_0_15_centos conf]# 
其中`victor.conf`和`wind.conf`中的内容如下：

	[root@VM_0_15_centos vhost]# ls
	victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com;
	        root /data/wwwroot/www.victor.com;
	}
	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	}
	[root@VM_0_15_centos vhost]# 

虚拟主机`www.victor.com`	的主目录内容如下：

	[root@VM_0_15_centos www.victor.com]# pwd
	/data/wwwroot/www.victor.com
	[root@VM_0_15_centos www.victor.com]# ls
	index.html
	[root@VM_0_15_centos www.victor.com]# cat index.html 
	This is www.victor.com!
	[root@VM_0_15_centos www.victor.com]# 

而`nginx.conf`中的配置如下：

	……
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	
	    sendfile        on;
	
	    keepalive_timeout  65;
	
	    include vhost/*.conf;
	
	}
	……
	
我们验证下访问这两个虚拟主机。

	# victor @ VICTORDONG-MB0 in ~ [22:57:41] 
	$ curl -H "Host:www.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [22:58:00] 
	$ curl -H "Host:www.wind.com" 150.109.76.79
	This is www.wind.com!
	
	# victor @ VICTORDONG-MB0 in ~ [22:58:08] 
	$ 
	
可以看到，我们通过指定http请求的header中的Host，可以访问到不同的主机。

通过抓包我们也可以看到，上面例子中两个请求其实就是http报文中请求头部的Host不一样。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nginx_vm1.png)

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nginx_vm2.png)


### 默认虚拟主机

可以设置一个默认虚拟主机。这样当访问的虚拟主机不存在时，就默认访问`默认虚拟主机`。


	[root@VM_0_15_centos vhost]# ls
	default.conf  victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	}
	[root@VM_0_15_centos vhost]# 
	
从上面可以看出，默认虚拟主机就是在`listen 80`后面添加一个`default_server`。
	
比如下面，主机`www.victor.comm`不存在，就访问到了默认虚拟主机。


	# victor @ VICTORDONG-MB0 in ~ [23:45:10] 
	$ curl -H "Host:www.victor.comm" 150.109.76.79
	default virtual machine
	
	# victor @ VICTORDONG-MB0 in ~ [23:45:14] 
	$ 

当然我们也可以针对默认虚拟主机进行拒绝访问。

默认虚拟主机配置如下：

	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        deny all;
	}
	[root@VM_0_15_centos vhost]# 

可以看到加了一行`deny all`。

我们测试下：

	# victor @ VICTORDONG-MB0 in ~ [23:45:14] 
	$ curl -H "Host:www.victor.comm" 150.109.76.79
	<html>
	<head><title>403 Forbidden</title></head>
	<body>
	<center><h1>403 Forbidden</h1></center>
	<hr><center>nginx/1.16.0</center>
	</body>
	</html>
	
	# victor @ VICTORDONG-MB0 in ~ [23:52:32] 
	$ 

直接返回403，符合预期。


### 虚拟主机index.html

如果我们不指定虚拟主机的index是哪个，那么nginx默认会去root 目录找名称为`index.html`的文件。

比如：

	[root@VM_0_15_centos vhost]# ls
	default.conf  victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com;
	        root /data/wwwroot/www.victor.com;
	}
	[root@VM_0_15_centos vhost]# cd /data/wwwroot/www.victor.com/
	[root@VM_0_15_centos www.victor.com]# ls
	index.html
	[root@VM_0_15_centos www.victor.com]# mv index.html index1.html
	[root@VM_0_15_centos www.victor.com]# ls
	index1.html
	[root@VM_0_15_centos www.victor.com]# 


虚拟主机`www.victor.com`没有指定index是哪个，root目录下面也没有名称为`index.html`的文件。那么我们访问虚拟主机`www.victor.com`会是啥结果呢？

	# victor @ VICTORDONG-MB0 in ~ [23:58:08] 
	$ curl -H "Host:www.victor.com" 150.109.76.79                                                                
	<html>
	<head><title>403 Forbidden</title></head>
	<body>
	<center><h1>403 Forbidden</h1></center>
	<hr><center>nginx/1.16.0</center>
	</body>
	</html>
	
	# victor @ VICTORDONG-MB0 in ~ [23:58:27] 
	$ 

可以看到是拒绝访问。

这时候，我们创建一个`index.html`文件。

	[root@VM_0_15_centos www.victor.com]# mv index1.html index.html
	[root@VM_0_15_centos www.victor.com]# ls
	index.html
	[root@VM_0_15_centos www.victor.com]# 
	
然后再访问虚拟主机`www.victor.com`。

可以看到正常访问。

	# victor @ VICTORDONG-MB0 in ~ [0:00:10] 
	$ curl -H "Host:www.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:00:11] 
	$ 

当然我们也可以指定`index`是哪个。

	[root@VM_0_15_centos www.victor.com]# cat /usr/local/nginx/conf/vhost/victor.conf                            
	server {
	        listen 80;
	        server_name www.victor.com;
	        index index1.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	}
	[root@VM_0_15_centos www.victor.com]# 
	
这里指定`index`为`index1.html`。

然后试下访问结果：

	# victor @ VICTORDONG-MB0 in ~ [0:35:11] 
	$ curl -H "Host:www.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:36:04] 
	$ 

可以看到正常访问到`index1.html`。

### 虚拟主机泛解析

有时候我们需要这样访问，`1.victor.com`，`2.victor.com`，希望访问这两个虚拟主机访问到同一个页面，可以这样做。


	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index1.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	}
	[root@VM_0_15_centos vhost]# 
	
通过在`server_name`处写上通配符`*.victor.com`就可以达到这样的目的。

验证结果如下：

	# victor @ VICTORDONG-MB0 in ~ [0:43:58] 
	$ curl -H "Host:1.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:44:03] 
	$ curl -H "Host:2.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:44:07] 
	$ curl -H "Host:www.victor.com" 150.109.76.79
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:44:12] 
	$ curl -H "Host:victor.com" 150.109.76.79 
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:44:16] 
	$ 

### 通过端口区分虚拟主机

上面我们谈到的都是通过`Host`来区分虚拟主机，还可以通过端口来区分虚拟主机。

比如下面这样：

	[root@VM_0_15_centos vhost]# cat victor_8080.conf 
	server {
	        listen 8080;
	        server_name www.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com_8080;
	}
	[root@VM_0_15_centos vhost]# 
	[root@VM_0_15_centos vhost]# cat /data/wwwroot/www.victor.com_8080/index.html 
	www.victor.com:8080
	[root@VM_0_15_centos vhost]# 
	
我们配置了`www.victor.com`的8080端口。

我们验证下：


	# victor @ VICTORDONG-MB0 in ~ [0:44:51] 
	$ curl -H "Host:victor.com" 150.109.76.79:8080
	www.victor.com:8080
	
	# victor @ VICTORDONG-MB0 in ~ [0:58:53] 
	$ curl -H "Host:victor.com" 150.109.76.79     
	This is www.victor.com!
	
	# victor @ VICTORDONG-MB0 in ~ [0:58:58] 
	$ 

可以看到，两者一个访问的是`/data/wwwroot/www.victor.com/index.html`，一个访问的是`/data/wwwroot/www.victor.com_8080/index.html`。

**应用场景**

有一种应用场景是，我们只有2台机器 A 和 B。希望通过A 和 B 一起给用户提供主机访问服务。我们可以在机器A上配置反向代理，代理地址为 www.xxxx.com:80。然后将请求转发到 机器A的的虚拟主机 hostA:8080 和机器B的虚拟主机 hostB:8080 上。

这样达到减少一台机器需求的目的。

## if ##

nginx 配置文件中可以使用`if`指令。

比如：

	if ($request_method = POST)
	{
	    return 405;
	}

逻辑判断符号有`=`，`~`，`~*`。

- `=`表示相等
- `~`表示正则匹配
- `~*`表示不区分大小写正则匹配


目标字符串可以是正则表达式，通常不用加引号（比如上面例子中的POST），但表达式中有特殊符号时，比如空格、花括号、分号等，需要用单引号引起来。
## rewrite

nginx 的`ngx_http_rewrite_module`模块给我们提供了一些很好用的功能。



- 域名跳转
- URL重写
- 动静分离

这个模块是自带的，不需要另外编译安装。先来看一个例子。

虚拟主机`www.victor.com`的配置如下：

	[root@VM_0_15_centos vhost]# ls
	default.conf  victor_8080.conf  victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        rewrite /1.html /2.html;
	        rewrite /2.html /3.html;
	}
	[root@VM_0_15_centos vhost]# 
	[root@VM_0_15_centos vhost]# cat /data/wwwroot/www.victor.com/1.html 
	This is www.victor.com! 1.html
	[root@VM_0_15_centos vhost]# cat /data/wwwroot/www.victor.com/2.html 
	This is www.victor.com! 2.html
	[root@VM_0_15_centos vhost]# cat /data/wwwroot/www.victor.com/3.html 
	This is www.victor.com! 3.html
	
重点关注`rewrite /1.html /2.html;`和`rewrite /2.html /3.html;`这两处。

我们验证下`rewrite`的功能。

	# victor @ VICTORDONG-MB0 in ~ [1:10:01] 
	$ curl -H "Host:victor.com" http://150.109.76.79/1.html
	This is www.victor.com! 3.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:10:09] 
	$ curl -H "Host:victor.com" http://150.109.76.79/2.html
	This is www.victor.com! 3.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:10:13] 
	$ curl -H "Host:victor.com" http://150.109.76.79/3.html
	This is www.victor.com! 3.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:10:16] 
	$                                                      

可以看到，我们请求`1.html`和`2.html`，最终都请求到了`3.html`。

我们查看`error.log`也可以看到这点。

	2019/05/28 01:13:53 [notice] 15868#0: *7 "/1.html" matches "/1.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                                   
	2019/05/28 01:13:53 [notice] 15868#0: *7 rewritten data: "/2.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                           
	2019/05/28 01:13:53 [notice] 15868#0: *7 "/2.html" matches "/2.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                                   
	2019/05/28 01:13:53 [notice] 15868#0: *7 rewritten data: "/3.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"     


这里跳转了2次。第一次跳转到`2.html`，然后`2.html`又匹配到了规则，`2.html`跳转到了`3.html`。

### 只匹配一次

可以通过`break`或者`last`来指定只匹配一次。比如虚拟主机`www.victor.com`的配置如下：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        rewrite /1.html /2.html break;
	        rewrite /2.html /3.html;
	}
	[root@VM_0_15_centos vhost]# 

重点关注`rewrite /1.html /2.html break;`这一行。

验证结果如下：

	# victor @ VICTORDONG-MB0 in ~ [1:16:39] 
	$ curl -H "Host:victor.com" http://150.109.76.79/1.html
	This is www.victor.com! 2.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:16:40] 
	$ 
	
可以看到只跳转到了`2.html`。

使用`last`的效果也是一样的。

### break和last的区别

前面讲到可以通过使用`break`和`last`来达到只匹配一次的效果，那么他们的区别是什么呢？

`break`和`last`在`server`里面的作用是一样的。

也就是下面这样配置：

	server {
		...
		rewrite xxx xxx break;
		rewrite xxx xxx last;
		...
	}
	

如果`break`和`last`是在`location`里面出现的话，他们是不一样的。

举个例子，虚拟主机`www.victor.com`的配置如下：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        location / {
	                rewrite /1.html /2.html;
	                rewrite /2.html /3.html;
	        }
	
	        location /2.html {
	                rewrite /2.html /b.html;
	        }
	        location /3.html {
	                rewrite /3.html /b.html;
	        }
	}
	[root@VM_0_15_centos vhost]# 
	
我们如果访问`1.html`结果会是如何呢？

	# victor @ VICTORDONG-MB0 in ~ [1:18:49] 
	$ curl -H "Host:victor.com" http://150.109.76.79/1.html                                                      
	<html>
	<head><title>404 Not Found</title></head>
	<body>
	<center><h1>404 Not Found</h1></center>
	<hr><center>nginx/1.16.0</center>
	</body>
	</html>
	
	# victor @ VICTORDONG-MB0 in ~ [1:35:40] 
	$ 

结果是`404`。

我们看下rewrite的日志。

	[root@VM_0_15_centos logs]# tail error.log
	2019/05/28 01:34:24 [notice] 20155#0: start worker process 20157
	2019/05/28 01:34:30 [notice] 20157#0: *1 "/1.html" matches "/1.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 rewritten data: "/2.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 "/2.html" matches "/2.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 rewritten data: "/3.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 "/3.html" matches "/3.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 rewritten data: "/b.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 "/1.html" does not match "/b.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [notice] 20157#0: *1 "/2.html" does not match "/b.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	2019/05/28 01:34:30 [error] 20157#0: *1 open() "/data/wwwroot/www.victor.com/b.html" failed (2: No such file or directory), client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	[root@VM_0_15_centos logs]# 

可以发现，先是匹配到了`/1.html`，然后跳转`/2.html`。然后`/2.html`继续被rewrite到了`/3.html`。这两步都是在`location / `里面完成的。

`/3.html`继续被匹配。可以匹配到`/3.html`的有两个location。分别是`location /`和`location /3.html`。nginx优先匹配更为精确的，所以`/3.html`被`location /3.html`匹配到了。然后被rewrite到了`/b.html`。

`/b.html`继续匹配过程。他可以被`location /`匹配到，但是里面的两个rewrite都匹配不到，所以最终就去访问`b.html`。由于`b.html`不存在，所以报错`404`。

从上面的rewrite log，我们可以也可以看出来。

我们也可以总结出一点：**rewrite从location里面出来之后，可以看做是一次新的请求，需要重新去匹配各个location**。


**使用break**

配置改一下，添加`break`。

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        location / {
	                rewrite /1.html /2.html break;
	                rewrite /2.html /3.html;
	        }
	
	        location /2.html {
	                rewrite /2.html /b.html;
	        }
	        location /3.html {
	                rewrite /3.html /b.html;
	        }
	}
	[root@VM_0_15_centos vhost]# 

然后验证：

	# victor @ VICTORDONG-MB0 in ~ [1:35:40] 
	$ curl -H "Host:victor.com" http://150.109.76.79/1.html                                                      
	This is www.victor.com! 2.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:47:13] 
	$ 
	
可以看到，访问的是`2.html`。

查看rewrite日志，也可以发现仅仅rewrite了一次。

	[root@VM_0_15_centos vhost]# tail ../../logs/error.log
	2019/05/28 01:47:11 [notice] 20155#0: exit
	2019/05/28 01:47:11 [notice] 21697#0: using the "epoll" event method                                         
	2019/05/28 01:47:11 [notice] 21697#0: nginx/1.16.0
	2019/05/28 01:47:11 [notice] 21697#0: built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)                   
	2019/05/28 01:47:11 [notice] 21697#0: OS: Linux 3.10.0-862.el7.x86_64                                        
	2019/05/28 01:47:11 [notice] 21697#0: getrlimit(RLIMIT_NOFILE): 1024:4096                                    
	2019/05/28 01:47:11 [notice] 21698#0: start worker processes                                                 
	2019/05/28 01:47:11 [notice] 21698#0: start worker process 21700                                             
	2019/05/28 01:47:13 [notice] 21700#0: *1 "/1.html" matches "/1.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                                   
	2019/05/28 01:47:13 [notice] 21700#0: *1 rewritten data: "/2.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                           
	[root@VM_0_15_centos vhost]#

这就是`break`的功效。

1. rewrite后面的rewrite也不执行了；
2. 跳出当前`location`，当前location后面的location也不去匹配了。

**使用last**

更改虚拟主机`www.victor.com`的配置：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        location / {
	                rewrite /1.html /2.html last;
	                rewrite /2.html /3.html;
	        }
	
	        location /2.html {
	                rewrite /2.html /b.html;
	        }
	        location /3.html {
	                rewrite /3.html /b.html;
	        }
	}
	[root@VM_0_15_centos vhost]# 
	
使用`last`。

验证：

	# victor @ VICTORDONG-MB0 in ~ [1:51:38] 
	$ curl -H "Host:victor.com" http://150.109.76.79/1.html                                                      
	<html>
	<head><title>404 Not Found</title></head>
	<body>
	<center><h1>404 Not Found</h1></center>
	<hr><center>nginx/1.16.0</center>
	</body>
	</html>
	
	# victor @ VICTORDONG-MB0 in ~ [1:51:48] 
	$ 
	
可以看到返回`404`。这是符合预期的。

查看下rewrite日志：


	[root@VM_0_15_centos vhost]# !tail
	tail ../../logs/error.log
	2019/05/28 01:51:43 [notice] 22372#0: getrlimit(RLIMIT_NOFILE): 1024:4096                                    
	2019/05/28 01:51:43 [notice] 22373#0: start worker processes                                                 
	2019/05/28 01:51:43 [notice] 22373#0: start worker process 22375                                             
	2019/05/28 01:51:47 [notice] 22375#0: *1 "/1.html" matches "/1.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                                   
	2019/05/28 01:51:47 [notice] 22375#0: *1 rewritten data: "/2.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                           
	2019/05/28 01:51:47 [notice] 22375#0: *1 "/2.html" matches "/2.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                                   
	2019/05/28 01:51:47 [notice] 22375#0: *1 rewritten data: "/b.html", args: "", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                           
	2019/05/28 01:51:47 [notice] 22375#0: *1 "/1.html" does not match "/b.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                            
	2019/05/28 01:51:47 [notice] 22375#0: *1 "/2.html" does not match "/b.html", client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"                                            
	2019/05/28 01:51:47 [error] 22375#0: *1 open() "/data/wwwroot/www.victor.com/b.html" failed (2: No such file or directory), client: 125.69.41.150, server: www.victor.com, request: "GET /1.html HTTP/1.1", host: "victor.com"
	[root@VM_0_15_centos vhost]#

可以看到，先是匹配到了locaion `location /`。然后执行里面的rewrite，将请求rewrite到了`/2.html`。

但是由于`last`的存在，不会去执行后面的rewrite，也就是不会去执行`rewrite /1.html /2.html last;`。

然后跳出当前location。

新请求`/2.html`继续匹配各个location，越精确越优先，所以匹配到了location `location /2.html`。执行里面的`rewrite /2.html /b.html;`，将请求rewrite到了`/b.html`，然后跳出当前location。

新请求`/b.html`继续匹配各个location。匹配到了location `location / `，但是没有匹配到里面的rewrite规则。所以最终去请求`/b.html`。

`b.html`不存在，所以报错`404`。


**结论**

- 当rewrite规则在location{}外，break和last作用一样，遇到break或last后，其后续的rewrite/return语句不再执行。但后续有location{}的话，还会近一步执行location{}里面的语句,当然前提是请求必须要匹配该location。
- 当rewrite规则在location{}里，遇到break后，本location{}与其他location{}的所有rewrite/return规则都不再执行。
- 当rewrite规则在location{}里，遇到last后，本location{}里后续rewrite/return规则不执行，但重写后的url再次从头开始执行所有规则，哪个匹配执行哪个。

### rewrite 规则 ###

- rewrite的语法格式为：

		rewrite regex replacement [flag]
- rewrite 配置可以在`if`，`server`，`location`配置段生效
- replacement是目标跳转的URI，可以是`http://`或者`https://`开头，也可以省略掉$host，直接写$request_uri部分
- flag用来设置rewrite对URI的处理行为，可以为`break`，`last`，`redirect`和`permanent`。`redirect`表示临时重定向，而`permanent`表示永久重定向

### rewrite 实战 ###

**访问二级目录**

比如在虚拟主机`www.victor.com`的主目录下建立一个子目录`sub`，sub目录下有文件`index.html`：

	[root@VM_0_15_centos sub]# pwd
	/data/wwwroot/www.victor.com/sub
	[root@VM_0_15_centos sub]# ls
	index.html
	[root@VM_0_15_centos sub]# 
	
虚拟主机`www.victor.com`的配置如下：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	
	        rewrite /(.*)   /sub/$1 last;
	}
	[root@VM_0_15_centos vhost]# 
	
然后我们验证访问二级目录：

	# victor @ VICTORDONG-MB0 in ~ [1:01:33] 
	$ curl -H "Host:victor.com" http://150.109.76.79:80/index.html   
	This is www.victor.com!
	sub directory
	
	# victor @ VICTORDONG-MB0 in ~ [1:01:34] 
	$ 
可以看到访问到了`/data/wwwroot/www.victor.com/sub/index.html`这个文件。




**静态请求分离**

有时候我们需要将静态请求和动态请求分离。将网站静态资源（html，js，img等文件）与后天应用分开部署，提高用户访问静态代码的速度。

举个例子：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        if ( $uri ~* 'jpg|jpeg|gif|css|png|js$')
	        {
	                rewrite /(.*) http://img.xxxxxx.com/$1 permanent;
	        }
	}
	[root@VM_0_15_centos vhost]# 

上面的配置是说，凡是请求`jpg,jpeg,gif,css,png,js`等后缀结尾的文件，全部重定向到`http://img.xxxxxx.com`主机上去请求。

**防盗链**

我们可以通过http请求头部的Referer请求行来判断请求的来源。如果这个来源不符合我们的预期，我们可以拒绝掉这个请求。这样就达到了防盗链的目的。

比如:

	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	
	        access_log  logs/wind.access.log;
	        location / {
	                valid_referers none blocked *.wind.com wind.com;
	                if ($invalid_referer)
	                {
	                        return 200 "forbidden";
	                }
	                echo "allowed";
	        }
	}
	[root@VM_0_15_centos vhost]# 
	
主机`www.wind.com`，只有当http请求中Referer字段符合特定要求时，才会返回`allowed`；否则返回`forbidden`。


`valid_referers none blocked *.wind.com wind.com;`指定了，只有当http请求中Referer字段符合以下几种要求时才会返回`allowed`。

- none，Referer不存在。
- blocked，Referer头部的值不以`http://`或者`https://`开头。
- *.wind.com，Referer头部的值形式为`http://*.wind.com`。
- wind.com，Referer头部的值形式为`http://wind.com`。

其他情况返回`forbidden`。

验证如下：

	# victor @ VICTORDONG-MB0 in ~ [1:05:05]                                       
	$ curl -H "Host:www.wind.com" http://hkcvm:80/ -v -e "https://h.wind.com"      
	*   Trying 124.156.182.201...
	* TCP_NODELAY set
	* Connected to hkcvm (124.156.182.201) port 80 (#0)                            
	> GET / HTTP/1.1
	> Host:www.wind.com
	> User-Agent: curl/7.54.0
	> Accept: */*
	> Referer: https://h.wind.com
	>
	< HTTP/1.1 200 OK
	< Server: nginx/1.16.0
	< Date: Mon, 10 Jun 2019 17:05:25 GMT
	< Content-Type: application/octet-stream                                       
	< Transfer-Encoding: chunked
	< Connection: keep-alive
	<
	allowed
	* Connection #0 to host hkcvm left intact                                      
	
	# victor @ VICTORDONG-MB0 in ~ [1:05:25]                                       
	$ 

forbidden的case如下：

	# victor @ VICTORDONG-MB0 in ~ [1:05:25]                                       
	$ curl -H "Host:www.wind.com" http://hkcvm:80/ -v -e "https://h.victor.com"    
	*   Trying 124.156.182.201...
	* TCP_NODELAY set
	* Connected to hkcvm (124.156.182.201) port 80 (#0)                            
	> GET / HTTP/1.1
	> Host:www.wind.com
	> User-Agent: curl/7.54.0
	> Accept: */*
	> Referer: https://h.victor.com
	>
	< HTTP/1.1 200 OK
	< Server: nginx/1.16.0
	< Date: Mon, 10 Jun 2019 17:08:17 GMT
	< Content-Type: application/octet-stream                                       
	< Content-Length: 9
	< Connection: keep-alive
	<
	* Connection #0 to host hkcvm left intact                                      
	forbidden%                                                                     
	
	# victor @ VICTORDONG-MB0 in ~ [1:08:18]                                       
	$ 
	
上面提到的`valid_referers`是一种nginx 指令。

**伪静态**

伪静态主要是为了增强搜索引擎的友好面。

比如discuz伪静态配置如下：

	location /  {
	    rewrite ^([^\.]*)/topic-(.+)\.html$ $1/portal.php?mod=topic&topic=$2 last;
	    rewrite ^([^\.]*)/forum-(\w+)-([0-9]+)\.html$ $1/forum.php?mod=forumdisplay&fid=$2&page=$3 last;
	    rewrite ^([^\.]*)/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=viewthread&tid=$2&extra=page%3D$4&page=$3 last;
	    rewrite ^([^\.]*)/group-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=group&fid=$2&page=$3 last;
	    rewrite ^([^\.]*)/space-(username|uid)-(.+)\.html$ $1/home.php?mod=space&$2=$3 last;
	    rewrite ^([^\.]*)/(fid|tid)-([0-9]+)\.html$ $1/index.php?action=$2&value=$3 last;
	}
	
用户是通过xxx.html链接来访问的，实际上却是访问到了动态网页。

**多个条件的并且**

有时候需要判断多个条件，当都满足时，然后去执行一定的操作。

nginx 配置文件 语法规则不支持`if`的嵌套。也就是说不支持这样：

	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        if ( $uri ~* 'jpg|jpeg|gif|css|png|js$')
	        {
	                if ($request_uri ~* '^/sub/victor.png')
	                {
	                        rewrite /(.*) /and.html last;
	                }
	        }
	}
	[root@VM_0_15_centos vhost]# service nginx configtest 
	nginx: [emerg] "if" directive is not allowed here in /usr/local/nginx/conf/vhost/victor.conf:11
	nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
	[root@VM_0_15_centos vhost]# 

但是我们可以这样：

	[root@VM_0_15_centos vhost]# cat victor.conf 
	server {
	        listen 80;
	        server_name www.victor.com victor.com *.victor.com;
	        index index.html
	        access_log  logs/victor.access.log;
	        root /data/wwwroot/www.victor.com;
	        rewrite_log on;
	        set $rule 0;
	        if ( $uri ~* 'jpg|jpeg|gif|css|png|js$')
	        {
	                set $rule "${rule}1";
	        }
	        if ($request_uri ~* '^/sub/victor.png')
	        {
	                set $rule "${rule}2";
	        }
	
	        if ($rule = "012")
	        {
	                rewrite /(.*) /and.html last;
	        }
	}
	[root@VM_0_15_centos vhost]# 
	
验证下：

	# victor @ VICTORDONG-MB0 in ~ [1:33:53] 
	$ curl -H "Host:victor.com" http://150.109.76.79:80/sub/victor.png 
	This is www.victor.com!
	and.html
	
	# victor @ VICTORDONG-MB0 in ~ [1:35:10] 
	$ 

我们最终访问到的是`/data/wwwroot/www.victor.com/and.html`。

	[root@VM_0_15_centos www.victor.com]# cat and.html 
	This is www.victor.com!
	and.html
	[root@VM_0_15_centos www.victor.com]# pwd
	/data/wwwroot/www.victor.com
	[root@VM_0_15_centos www.victor.com]# 

这里我们通过控制变量`$rule`的值，来达到让多个条件取`&&`的效果。


## nginx模块 ##

nginx 可以通过模块来丰富他的功能。这里以[echo-nginx-module](https://github.com/openresty/echo-nginx-module#installation)为例。

### 安装 ###
1. 下载`echo-nginx-module`模块的源码

		https://github.com/openresty/echo-nginx-module.git
2. 重新编译nginx

		[root@VM_0_15_centos nginx-1.16.0]# ls
		auto     CHANGES.ru  configure  html     Makefile  objs    src
		CHANGES  conf        contrib    LICENSE  man       README
		[root@VM_0_15_centos nginx-1.16.0]# pwd
		/root/src/nginx-1.16.0
		[root@VM_0_15_centos nginx-1.16.0]# ./configure --prefix=/usr/local/nginx --add-
		module=/root/src/echo-nginx-module
	其中，`/root/src/echo-nginx-module`是`echo-nginx-module`模块的源码路径。
3. make
4. make install

总结一下，就是需要重新编译nginx，然后安装。

最好事先将nginx进程停止。

	service nginx stop
	
### 使用 ###
以虚拟主机`www.wind.com`为例，先看下虚拟主机的配置：

	[root@VM_0_15_centos vhost]# ls
	default.conf  victor_8080.conf  victor.conf  wind.conf
	[root@VM_0_15_centos vhost]# pwd
	/usr/local/nginx/conf/vhost
	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	        location / {
	                echo "Test echo module";
	        }
	}
	[root@VM_0_15_centos vhost]# 

验证一下：

	# victor @ VICTORDONG-MB0 in ~ [2:12:40] 
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80           
	Test echo module
	
	# victor @ VICTORDONG-MB0 in ~ [2:12:43] 
	$ 

可以看到直接返回了`echo`指令后面的内容。而并没有返回`/data/wwwroot/www.wind.com/index.html`里面的内容。
	
	[root@VM_0_15_centos vhost]# ls -l /data/wwwroot/www.wind.com/index.html 
	-rw-r--r-- 1 root root 22 5月  26 22:28 /data/wwwroot/www.wind.com/index.html
	[root@VM_0_15_centos vhost]# cat /data/wwwroot/www.wind.com/index.html 
	This is www.wind.com!
	[root@VM_0_15_centos vhost]# 
	
如果我们把echo指令去掉，像下面这样，

	[root@VM_0_15_centos vhost]# ls
	default.conf  victor_8080.conf  victor.conf  wind.conf                         
	[root@VM_0_15_centos vhost]# vim wind.conf                                     
	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	        location / {
	        }
	}
	[root@VM_0_15_centos vhost]# 

那么：


	# victor @ VICTORDONG-MB0 in ~ [2:15:41] 
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80
	This is www.wind.com!
	
	# victor @ VICTORDONG-MB0 in ~ [2:15:42] 
	$ 
	
可以看到返回的就是`index.html`的内容。

总结一下：`echo`会屏蔽掉请求的文件，比如上面的`index.html`。

## nginx变量 ##
上面提到了，我们可以在配置文件中设置变量的值。



在nginx配置中，变量只能存放一种类型的值，有且也只存在一种类型，那就是字符串类型。

比如上面我们用到的`set $rule 0;`，是使用ngx_rewrite模块的`set`配置指令对变量`$rule`进行了赋值操作。我们把字符串`0`赋值给了`$rule`变量。

所有的nginx变量在nginx配置文件中必须带上`$`符号前缀。这种表示方法的好处是显而易见的，那就是可以把变量嵌入到字符串常量中，以构造出新的字符串。比如：

	set $a hello;
	set $b "$a, $a";

执行完后，`$b`的值是`hello, hello`。

这种技术在perl中被称为**变量插值**，它让专门的字符串拼接运算符变得不再那么必要。


需要说明的是，并不是所有的配置指令都支持**变量插值**。事实上，指令参数是否支持变量插值，取决于该指令的实现模块。比如上面谈到的`echo-nginx-module`模块是支持**变量插值**的。

那如果想直接输出美元符号`$`，有办法转义么？	

目前没有办法！

不过可以通过不支持**变量插值**的模块配置指令构造出取值为`$`的nginx变量，然后再在echo中使用这个变量。比如：

	[root@VM_0_15_centos vhost]# cat wind.conf 
	geo $dollar {
	        default "$";
	}
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	        location / {
	                echo "This is a dollar sign: $dollar";
	        }
	}
	[root@VM_0_15_centos vhost]# 
	
验证如下：

	# victor @ VICTORDONG-MB0 in ~ [9:45:29]                                       
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80/                         
	This is a dollar sign: $
	
	# victor @ VICTORDONG-MB0 in ~ [9:47:39]                                       
	$ 



在**变量插值**的case中，当引用的变量名后面跟着其他字符时，我们需要通过`{}`来消除歧义，比如：

	server {  
	  listen 8080;  
	 
	  location /test {  
	    set $first "hello ";  
	    echo "${first}world";  
	  }  
	} 

这里我们需要给`first`变量加上`{}`括起来，不然nginx会误认为引用了变量`$firstworld`。

### set变量 ###

`set`指令不仅有赋值的功能，还有创建nginx变量的副作用。即当作为赋值对象的变量尚不存在时，它会自动创建改变量。

比如上面提到的`$first`变量，`set`指令创建了`first`变量。

如果我们使用不存在的变量，那么nginx就启动不起来。

### 变量何时创建 ###

nginx变量的创建和赋值发生在完全不同的时间段。

创建只能发生在nginx配置加载的时候，而赋值操作则只会发生在请求实际处理的时候。

这意味着不创建而直接使用变量会导致启动失败，同时也意味着我们无法在请求处理时动态地创建新的 nginx 变量。

nginx 变量一旦创建，其变量名的可见范围就是整个 nginx 配置，甚至可以跨越不同虚拟主机的 server 配置块。

比如下面：

	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	        location / {
	                echo "var=$duck";
	        }
	        location /var {
	                set $duck "duck";
	                echo "var=$duck";
	        }
	}
	[root@VM_0_15_centos vhost]# 

变量`duck`的创建是在`/var`中，但是在`/`中也是可以使用变量`duck`的。


验证如下：

	# victor @ VICTORDONG-MB0 in ~ [18:30:24]                                      
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80/
	var=
	
	# victor @ VICTORDONG-MB0 in ~ [18:35:11] 
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80/var
	var=duck
	
	# victor @ VICTORDONG-MB0 in ~ [18:35:15] 
	$ 


可以发现，请求`/var`接口时，我们得到了duck；请求`/`接口时，我们得到的是空字符串，因为用户变量未赋值就输出的话，得到的便是空字符串。

我们还可以窥见的另一个重要特性是，nginx 变量名的可见范围虽然是整个配置，但每个请求都有所有变量的独立副本，彼此互不干扰。

nginx 变量的生命期是不可能跨越请求边界的。

### 变量的生命周期 ###

nginx的变量生命周期是是与请求相关的，而不是与location相关。

比如：

	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	
	        access_log  logs/wind.access.log;
	        location / {
	                set $duck "duck";
	                echo_exec /var;
	        }
	
	        location /var {
	                echo "var=$duck";
	        }
	}
	[root@VM_0_15_centos vhost]# 

验证如下：

	# victor @ VICTORDONG-MB0 in ~ [1:18:09] 
	$ curl -H "Host:www.wind.com" http://119.28.68.178:80/
	var=duck
	
	# victor @ VICTORDONG-MB0 in ~ [1:18:10] 
	$ 

可以看到，我们访问的location是`/`，在location`/`中，我们创建了变量`$duck`，并设置了值为duck。

然后通过`echo_exec`跳转到了`/var`，输出变量`$duck`的值为duck。

变量`$duck`的值跨越了location。

## return ##

可以直接在`server`中reutrn响应的状态码、字符串或者url。在该作用域内return后面的所有nginx配置都是无效的。

可以使用在server、location以及if配置中。

### return url ###
比如默认虚拟主机配置如下：

	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        return http://www.baidu.com;
	}
	[root@VM_0_15_centos vhost]# 
	
我们在电脑浏览器访问`150.109.76.79`，就被直接跳转到了百度的首页。（150.109.76.79是我们nginx服务部署的机器IP）。

return url 可以达到域名跳转的功能。

这里需要注意的是，在return url的时候，url前面不能加状态码，比如`return 200 http://www.baidu.com`这种，这样返回的就是字符串了。

如果加状态码的话，也是`return 301 http://www.baidu.com`或者`return 302 http://www.baidu.com`这种。

这是因为301（永久重定向）和302（临时重定向）都表示重定向。

我们也可以使用rewrite来达到同样的目的。比如：


	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        rewrite /(.*) www.baidu.com;
	}
	[root@VM_0_15_centos vhost]# 

同样可以达到跳转的目的。

### return code ###

返回http 状态码。比如：

	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        return 404;
	}
	[root@VM_0_15_centos vhost]# 

那么我们在浏览器上访问`150.109.76.79`，就会显示`404 Not Found`。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/404.png)

### return 字符串 ###

default虚拟主机如下：

	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        return 200 "This is www.default.com";
	}
	[root@VM_0_15_centos vhost]# 


 验证如下：
 
	# victor @ VICTORDONG-MB0 in ~ [0:41:28] 
	$ curl http://150.109.76.79
	This is www.default.com%                                                                                      
	
	# victor @ VICTORDONG-MB0 in ~ [0:53:57] 
	$ 
### return 变量 ###

也可以return nginx的变量。比如：

	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 80 default_server;
	        root /data/wwwroot/www.default.com;
	        return 200 "$host $request_uri";
	}
	[root@VM_0_15_centos vhost]# 

验证如下：

	# victor @ VICTORDONG-MB0 in ~ [0:56:33] 
	$ curl http://150.109.76.79
	150.109.76.79 /%                                                                                              
	
	# victor @ VICTORDONG-MB0 in ~ [0:56:40] 
	$ 
	
## nginx常用变量 ##

nginx常用全局变量如下表：

| 变量       | 说明    |
| :--------   | :-----   | 
|$args       |请求中的参数，如www.123.com/1.php?a=1&b=2的$args就是a=1&b=2 |
|$content_length |HTTP请求信息里的"Content-Length" |
|$conten_type    |  HTTP请求信息里的"Content-Type"   |
|$document_root|nginx虚拟主机配置文件中的root参数对应的值|
|$document_uri|当前请求中不包含指令的URI，如www.123.com/1.php?a=1&b=2的$document_uri就是1.php,不包含后面的参数|
|$host|主机头，也就是域名|
|$http_user_agent|客户端的详细信息，也就是浏览器的标识，用curl -A可以指定|
|$http_cookie|客户端的cookie信息|
|$limit_rate|如果nginx服务器使用limit_rate配置了显示网络速率，则会显示，如果没有设置， 则显示0|
|$remote_addr|客户端的公网ip|
|$remote_port|客户端的port|
|$remote_user|如果nginx有配置认证，该变量代表客户端认证的用户名|
|$request_body_file|做反向代理时发给后端服务器的本地资源的名称|
|$request_method|请求资源的方式，GET/PUT/DELETE等|
|$request_filename|当前请求的资源文件的路径名称，相当于是$document_root/$document_uri的组合|
|$request_uri|请求的链接，包括$document_uri和$args|
|$scheme|请求的协议，如ftp,http,https|
|$server_protocol|客户端请求资源使用的协议的版本，如HTTP/1.0，HTTP/1.1，HTTP/2.0等|
|$server_addr|服务器IP地址|
|$server_name|服务器的主机名|
|$server_port|服务器的端口号|
|$uri|和$document_uri相同|
|$http_referer|客户端请求时的referer，通俗讲就是该请求是通过哪个链接跳过来的，用curl -e可以指定|

## location ##

nginx location 的语法规则是:

	location [=|~|~*|^~] /uri/ {
	}

location 匹配的变量是`$uri`。

规则优先级：

`=`高于`^~`高于`~*`等于`~`高于`/`。



比如虚拟主机`www.wind.com`配置如下：

	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 80;
	        server_name www.wind.com;
	        root /data/wwwroot/www.wind.com;
	
	        access_log  logs/wind.access.log;
	        location / {
	                echo "/";
	        }
	        location ~ abc {
	                echo "~";
	        }
	        location ^~ /abc {
	                echo "^~";
	        }
	}
	[root@VM_0_15_centos vhost]# 

验证如下：

	# victor @ VICTORDONG-MB0 in ~ [1:06:35]                                       
	$ curl -H "Host:www.wind.com" http://hkcvm:80/dabcd
	~
	
	# victor @ VICTORDONG-MB0 in ~ [1:08:14] 
	$ curl -H "Host:www.wind.com" http://hkcvm:80/abc  
	^~
	
	# victor @ VICTORDONG-MB0 in ~ [1:08:19] 
	$ curl -H "Host:www.wind.com" http://hkcvm:80/axbc
	/
	
	# victor @ VICTORDONG-MB0 in ~ [1:08:30] 
	$ 
	
需要注意的点，location不支持`!~`。

也就是下面这样是非法的：

	#非法的
	location !~ abc {
	        echo "~";
	}

	
	
## 正向代理 ##

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/z_proxy.png)

上面这张图说明了正向代理是咋回事。

**请求方**告诉**代理服务器**，你去帮我从**web服务器**那里请求某个内容。**代理服务器**从**web服务器**那里取得内容之后，再返回给**请求方**。

nginx 可以通过几行简单配置，实现正向代理。

	[root@VM_0_15_centos vhost]# cat proxy.conf 
	server {
	        listen 80 default_server;
	        resolver 119.29.29.29;
	        access_log  logs/proxy.access.log;
	        location / {
	                proxy_pass http://$host$request_uri;
	        }
	}
	[root@VM_0_15_centos vhost]# 


上面`resolver 119.29.29.29`表示DNS服务器的地址。`119.29.29.29`为DNSPOD提供的DNS服务器。


## 反向代理 ##

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/f_proxy.png)


上面这张图说明了反向代理是咋回事。

**请求方**是不知道代理服务器后面隐藏的**WEB服务器**的，它以为**代理服务器**就是真正的内容提供方。

利用nginx可以方便地搭建反向代理服务器。

举个例子。

我们建立了3个虚拟主机，分别是`www.wind.com:80`，`www.wind.com:8080`和在8080端口监听的`default_server`。

	[root@VM_0_15_centos vhost]# ls
	default.conf  fproxy.conf  wind.conf
	[root@VM_0_15_centos vhost]# cat default.conf 
	server {
	        listen 8080 default_server;
	        location / {
	                echo "default_server 8080";
	        }
	}
	[root@VM_0_15_centos vhost]# cat fproxy.conf 
	server {
	        listen 80;
	        server_name     www.wind.com wind.com;
	        access_log  logs/proxy.access.log;
	        location / {
	                proxy_pass http://127.0.0.1:8080/;
	                proxy_set_header Host $host;
	                proxy_set_header X-Real-IP $remote_addr;
	                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        }
	}
	[root@VM_0_15_centos vhost]# 
	[root@VM_0_15_centos vhost]# cat wind.conf 
	server {
	        listen 8080;
	        server_name     www.wind.com wind.com;
	        location / {
	                echo "www.wind.com 8080";
	        }
	}
	[root@VM_0_15_centos vhost]# 



其中`fproxy.conf`表示反向代理服务器的配置。

我们验证一下：


	# victor @ VICTORDONG-MB0 in ~ [1:02:36] 
	$ curl -H "Host:www.wind.com" http://hkcvm:8080   
	www.wind.com 8080
	
	# victor @ VICTORDONG-MB0 in ~ [1:02:37] 
	$ curl http://hkcvm:8080 
	default_server 8080
	
	# victor @ VICTORDONG-MB0 in ~ [1:02:48] 
	$ 
	
可以看到，如果不加Host，那么访问到了8080端口的default_server；如果加了Host，那么访问到的是`www.wind.com:8080`虚拟主机。

那如果我们访问`http://hkcvm`地址，而不是访问`http://hkcvm:8080`呢？

验证一下：

	# victor @ VICTORDONG-MB0 in ~ [1:02:48] 
	$ curl -H "Host:www.wind.com" http://hkcvm     
	www.wind.com 8080
	
	# victor @ VICTORDONG-MB0 in ~ [1:05:21] 
	$ 
	
可以看到，最终访问到了`www.wind.com:8080`虚拟主机的内容。为什么呢？

我们再来看下proxy的配置：

	server {
	        listen 80;
	        server_name     www.wind.com wind.com;
	        access_log  logs/proxy.access.log;
	        location / {
	                proxy_pass http://127.0.0.1:8080/;
	                proxy_set_header Host $host;
	                proxy_set_header X-Real-IP $remote_addr;
	                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        }
	}
	
`curl -H "Host:www.wind.com" http://hkcvm`，此请求在http报文请求行中标注了Host为`www.wind.com`，表示其想访问的是80端口监听的`www.wind.com`主机。

而实际上80端口监听的`www.wind.com`主机是一个反向代理服务器，这个代理服务器偷偷摸摸地去请求http://127.0.0.1:8080端口监听的`$host`主机，此处`$host`取值为`www.wind.com`。

也就是说代理服务器背地里类似`curl -H "Host:www.wind.com" http://127.0.0.1:8080/`这样发了次请求，然后把返回的内容吐还给最初的请求方。这个内容就是`www.wind.com 8080`。

**可以看到请求方是不知道代理服务器背地里干了啥事的。**

为了进一步验证，如果我们把上面`fproxy.conf`中的`proxy_set_header Host $host;`去掉，会怎么样呢？

	[root@VM_0_15_centos vhost]# cat fproxy.conf 
	server {
	        listen 80;
	        server_name     www.wind.com;
	        access_log  logs/proxy.access.log;
	        location / {
	                proxy_pass http://127.0.0.1:8080/;
	                proxy_set_header X-Real-IP $remote_addr;
	                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        }
	}
	[root@VM_0_15_centos vhost]# 

验证如下：

	# victor @ VICTORDONG-MB0 in ~ [1:32:57] 
	$ curl http://wind.com   
	default_server 8080
	
	# victor @ VICTORDONG-MB0 in ~ [1:32:59] 
	$ 
	
可以看到，最终访问到了8080端口监听的default_server。

原因是由于代理服务器背地里去请求真正内容提供方时候，没有指定Host，你可以认为他是这么干的：

	curl http://127.0.0.1:8080/

这个会返回什么呢？

	[root@VM_0_15_centos vhost]# curl http://127.0.0.1:8080/
	default_server 8080
	[root@VM_0_15_centos vhost]# 

与上面在mac上使用`curl http://wind.com `请求的结果一致。


### 区别 ###

正向代理和方向代理，其实本质上都是nginx代替请求方去请求真正的资源，nginx不是真正的内容提供方。

区别在于，对于反向代理，用户是不知道他的请求目的方是代理的，他以为他请求的就是真正内容的提供方。所以他在http请求时候，http报文的请求行中，Host字段一般填的就是代理服务器的server\_name。

而对于正向代理，用户是知道这个代理的存在的。那么我在请求http的报文中，Host字段就不会填你代理服务器的server\_name，而是填上了我最终想访问的虚拟主机的server\_name。然后正向代理服务器根据这个server\_name去代替我去请求。


其实我们从proxy\_pass这个nginx指令也可以看出，正向代理和反向代理，nginx干的事情几乎一样，都是通过proxy\_pass指令来实现的。