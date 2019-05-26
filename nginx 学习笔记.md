# nginx 学习笔记 #
# 安装
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
		
8. 如果想要开机启动可以使用`chkconfig`，`chkconfig --add nginx`，`chkconfig nginx --level 4 on`。

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
