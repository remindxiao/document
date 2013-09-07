Nginx 安装指南
=============

------------------------

## 环境参数

* 操作系统版本: 

		Linux version 2.6.32-220.el6.x86_64 (mockbuild@x86-004.build.bos.redhat.com) 
		(gcc version 4.4.5 20110214 (Red Hat 4.4.5-6) (GCC) 

* 依赖条件:
	
	+ gzip module requires zlib library
	+ rewrite module requires pcre library
	+ ssl support requires openssl library

################################# 

* 推荐工具

	+ tmux
	+ secureCRT
	+ winscp


## 安装

* 下载 [[zlib-1.2.8.tar.gz](http://prdownloads.sourceforge.net/libpng/zlib-1.2.8.tar.gz?download)]

		sudo wget http://prdownloads.sourceforge.net/libpng/zlib-1.2.8.tar.gz?download

* 下载 [[pcre-8.33.tar.gz](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.33.tar.gz)]

		sudo wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.33.tar.gz

* 「可选」下载 

* 下载 [[nginx-1.4.2.tar.gz](http://nginx.org/download/nginx-1.4.2.tar.gz)]

		sudo wget http://nginx.org/download/nginx-1.4.2.tar.gz


* 解压文件

		tar zxf zlib-1.2.8.tar.gz
		tar zxf pcre-8.33.tar.gz
		tar zxf nginx-1.4.2.tar.gz
	
* 移动文件到安装路径(示例中为`/opt目录下`)

		sudo mv zlib-1.2.8 /opt
		sudo mv pcre-8.33 /opt
		sudo mv nginx-1.4.2 /opt

* 编译`nginx`

		cd /opt/nginx-1.4.2/
		./configure --prefix=/usr/local/ --sbin-path=/usr/local/sbin --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/run/nginx/client_body_temp --http-proxy-temp-path=/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/var/run/nginx/scgi_temp --with-pcre=../pcre-8.33 --with-zlib=../zlib-1.2.8 --with-openssl=../openssl-1.0.1e --with-ipv6 --with-http_ssl_module --with-http_gzip_static_module


	**参数说明**
 		
	路径	
	
	+ `--prefix=/usr/local/`
	+ `--sbin-path=/usr/local/sbin`
	+ `--conf-path=/etc/nginx/nginx.conf`
	+ `--pid-path=/var/run/nginx.pid`
	+ `--lock-path=/var/run/nginx.lock`
	+ `--error-log-path=/var/log/nginx/error.log`
	+ `--http-log-path=/var/log/nginx/access.log`
	
	临时文件

	+ `--http-client-body-temp-path=/var/run/nginx/client_body_temp`
	+ `--http-proxy-temp-path=/var/run/nginx/proxy_temp` 
	+ `--http-fastcgi-temp-path=/var/run/nginx/fastcgi_temp` 
	+ `--http-uwsgi-temp-path=/var/run/nginx/uwsgi_temp` 
	+ `--http-scgi-temp-path=/var/run/nginx/scgi_temp`
	
	依赖
	
	+ `--with-pcre=../pcre-8.33` 
	+ `--with-zlib=../zlib-1.2.8` 
	+ `--with-openssl=../openssl-1.0.1e` 
	
	功能模块

	+ `--with-ipv6` 
	+ `--with-http_ssl_module` 
	+ `--with-http_gzip_static_module`
	
	详情请参考
	
	[[Installation and compile-time options](http://wiki.nginx.org/NginxInstallOptions)]

		
* 安装
	
		make & make install

	
* 问题一： nginx安装执行 `make` 时报 `You need a C++ compiler for C++ support`

	解决： 安装 gcc-c++ 工具
	
		sudo yum install gcc*




## 配置

	
* 创建nginx用户

		# 创建用户，但不创建用户 home
		useradd -M nginx
		# 锁定用户，使用户无法通过 password 登录
		usermod -L nginx

* 配置nginx文件

	创建 `sites-availiable` 目录
		
		mkdir /etc/nginx/sites-availiable

	创建 `sites-enabled` 目录

		mkdir /etc/nginx/sites-enabled

	创建 虚拟主机
	
		vim /etc/nginx/sites-availiable/web.conf

	`web.conf` 内容如下

		upstream web_server {
			# WebContainer Server Port
		    server localhost:8888;
		}
		
		server {
				# if url is yourdomain.com,then redirect to www.yourdomain.com
		        listen 7002;
		        server_name yourdomain.com ;
		        return 301 http://www.yourdomain.com$request_uri;
		}
		
		server {
			# Listen Port for nginx
		    listen 7002 default_server;
		    server_name  www.yourdomain.com;
		
		    root /home/admin/project/nuwa;
		
		    # individual nginx logs for this web vhost
		    access_log  /var/log/nginx/www.yourdomain.com/access.log;
		    error_log   /var/log/nginx/www.yourdomain.com/error.log ;
		    
		    location = /favicon.ico {
		       return  404;
		    }
		    
		    location ~ /(.*)\.html {
		        break;
		    }
		    
		    location = /robots.txt {
		        alias /usr/local/nginx/html/robots.txt;
		    }
		
		    #when not specify request uri, redirect to /index;
		    location = / {
		       rewrite ^ /index ;
		    } 
		
		    
		    #server assets
		    location ~ ^/(assets|templates)/(.*)$ {
		       try_files $uri $uri/;
		    }
		
		    location ~ ^/(design|api|images)/(.*)$ {
		       proxy_pass              http://web_server;
		       proxy_set_header        X-Real-IP $remote_addr;
		       proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		       proxy_set_header        Host $http_host;
		    }
		    
		    
		    #1.www.yourdomain.com/index --> www.yourdomain.com/sites/www.yourdomain.com/index
		    #2.aaa.yourdomain.com/index --> www.yourdomain.com/sites/aaa.yourdomain.com/index
		    location / {
		       rewrite ^(.*)$ /sites/$host$1 break;
		       proxy_pass http://web_server;
		       proxy_set_header        X-Real-IP $remote_addr;
		       proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		       proxy_set_header        Host $http_host;
		    }
		}
		
	若存在多个服务，则再创建如 `admin.conf`
	
		vim /etc/nginx/sites-availiable/admin.conf

	`admin.conf` 内容如下
		
		upstream back_server {
		    server localhost:8888;
		}
		
		server {
		    listen 7002 ;
		    server_name  admin.yourdomain.com ;
		
		    root /home/admin/project/fuhsi/;
		
		    # individual nginx logs for this web vhost
		    access_log  /var/log/nginx/admin.yourdomain.com/access.log;
		    error_log   /var/log/nginx/admin.yourdomain.com/error.log ;
		    
		    #when not specify request uri, redirect to /index;
		    location = / {
		        rewrite ^ /components;
		    } 
		
		    location = /favicon.ico {
		       return  404;
		    }
		    
		    location ~ /(.*)\.html {
		           break;
		    }
		    
		    #server assets
		    location ~ ^/(assets|templates)/(.*)$ {
		       try_files $uri $uri/;
		    }
		
		    #if request path starts with /admin, then just pass to back server
		    location ~ ^/admin/(.*)$ {
		       proxy_pass http://back_server;
		       proxy_set_header        X-Real-IP $remote_addr;
		       proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		       proxy_set_header        Host $http_host; 
		    }
		
		    # add /admin to request uri; 
		    location / {
		       rewrite ^(.*)$ /admin$1 break;
		       proxy_pass http://back_server;
		       proxy_set_header        X-Real-IP $remote_addr;
		       proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		       proxy_set_header        Host $http_host;
		    }
		}	
	



* 安装服务
	
	在 `/etc/init.d/`创建 `nginx` 启动脚本

		sudo vim /etc/init.d/nginx
	
	脚本内容如下：
		
		# nginx - this script starts and stops the nginx daemon
		#
		# chkconfig:   - 85 15 
		# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
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
		 
		nginx="/usr/bin/nginx"
		prog=$(basename $nginx)
		 
		NGINX_CONF_FILE="/etc/nginx/nginx.conf"
		 
		[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
		 
		lockfile=/var/run/nginx.lock
		 
		make_dirs() {
		   # make required directories
		   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
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
				
	配置服务
	
	
		# 增加服务启动项
		sudo chkconfig --add nginx
		# 设置开机自动启动
		sudo chkconfig --level 345 nginx on
		# 赋予 nginx 可执行权限
		sudo chmod +x nginx



## 参考

* [[Getting Started with Nginx](http://wiki.nginx.org/GettingStarted)]
* [[FullExample](http://wiki.nginx.org/FullExample)]
* [[List of core configuration directives](http://wiki.nginx.org/Configuration)]
* [[Red Hat Nginx Init Script](http://wiki.nginx.org/RedHatNginxInitScript)]