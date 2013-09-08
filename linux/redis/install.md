![](http://redis.io/images/redis-small.png) Redis-Server 安装指南
====================



---------------------------


## 安装
	
* 下载

	wget http://download.redis.io/releases/redis-2.6.16.tar.gz

* 解压编译

	tar xvzf redis-2.6.16.tar.gz
	mv redis-2.6.16 /usr/local/
	cd redis-stable
	make

* 执行

	make test

* 问题: `make test` 时报 `You need tcl 8.5 or newer in order to run the Redis test`

	解决:安装 `tcl` 8.5 以上版本

		sudo yum install tcl


* 执行完单元测试后，在 `src` 目录下会生成

	+ **redis-server**
	+ **redis-cli**

	两个文件，然后执行
	
		cd /usr/local/redis-2.6.16/src
		sudo cp redis-server /usr/local/bin
		sudo cp redis-cli /usr/local/bin


* 启动 `redis-server`

	redis-server

* 测试联通性

		redis-cli                                                                
		redis 127.0.0.1:6379> ping
	
	>PONG
	
		redis 127.0.0.1:6379> set mykey somevalue
		
	>OK
		
		redis 127.0.0.1:6379> get mykey
		
	>"somevalue"


* 配置服务
		
		# create config folder
		sudo mkdir /etc/redis
		# create data folder
		sudo mkdir /var/redis

* 复制 `redis` 开机启动脚本

		cd /etc/init.d/
		sudo wget https://raw.github.com/gist/257849/9f1e627e0b7dbe68882fa2b7bdb1b2b263522004/redis-server
		sudo mv redis-server redis_6379

* 修改脚本

		sudo vim /etc/init.d/redis_6379

	内容如下:

		#!/bin/sh
		#
		# redis - this script starts and stops the redis-server daemon
		#
		# chkconfig:   - 85 15 
		# description:  Redis is a persistent key-value database
		# processname: redis-server
		# config:      /etc/redis/redis.conf
		# config:      /etc/sysconfig/redis
		# pidfile:     /var/run/redis.pid
		
		# Source function library.
		. /etc/rc.d/init.d/functions
		
		# Source networking configuration.
		. /etc/sysconfig/network
		
		# Check that networking is up.
		[ "$NETWORKING" = "no" ] && exit 0
		
		redis="/usr/local/bin/redis-server"
		prog=$(basename $redis)
		
		REDIS_CONF_FILE="/etc/redis/6379.conf"
		
		[ -f /etc/sysconfig/redis ] && . /etc/sysconfig/redis
		
		lockfile=/var/lock/subsys/redis_6379
		
		start() {
		    [ -x $redis ] || exit 5
		    [ -f $REDIS_CONF_FILE ] || exit 6
		    echo -n $"Starting $prog: "
		    daemon $redis $REDIS_CONF_FILE
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
		    stop
		    start
		}
		
		reload() {
		    echo -n $"Reloading $prog: "
		    killproc $redis -HUP
		    RETVAL=$?
		    echo
		}
		
		force_reload() {
		    restart
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
		        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
		        exit 2
		esac



* 复制并修改 `redis.conf`

		cd $REDIS_HOME
		sudo cp redis.conf /etc/redis/6379.conf

	创建`redis`实例数据存放目录

		sudo mkdir /var/redis/6379

	修改配置文件

		sudo vim /etc/redis/6379.conf

	内容如下(仅列出重要配置)：

	- Set daemonize to yes (by default it is set to no). 
		
		以守护进程的方式启动

			daemonize yes 
	
	- 	Set the pidfile to /var/run/redis_6379.pid (modify the port if needed). 
	
		进程文件路径
	
			pidfile /var/run/redis_6379.pid

	- 	Change the port accordingly. In our example it is not needed as the default port is already 6379. 
		
		启动端口号

			port 6379

	- 	Set your preferred loglevel.
	
		日志记录等级 `debug` | `verbose` | `notice` | `warning`

			loglevel notice
		
	- 	Set the logfile to /var/log/redis_6379.log 
		
		日志路径

			logfile /var/log/redis/redis_6379.log 

	- 	Set the dir to /var/redis/6379 (very important step!) 
	
		数据文件存放目录

			dir /var/redis/6379  
	

* 配置服务

		chmod 755 /etc/init.d/redis_6379
		chkconfig --add redis_6379
		chkconfig --level 345 redis_6379 on

* 优化服务器配置，为什么这样做？ 请看「[这里](http://www.saltwebsites.com/2012/install-redis-245-service-centos-6)」

		sudo vim /etc/sysctl.conf

	设置（若不存在则新增）

		vm.overcommit_memory = 1

	使配置生效	
	
		sudo sysctl vm.overcommit_memory=1

	启动服务

		service redis-server start


## 参考

* [Redis Quick Start](http://redis.io/topics/quickstart)
* [Install Redis 2.4.5 as Service on Centos 6](http://www.saltwebsites.com/2012/install-redis-245-service-centos-6)


## 备注

**overcommit_memory** 文件指定了内核针对内存分配的策略，其值可以是 **0**、**1**、**2**。

* 0： 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
* 1: 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
* 2: 表示内核允许分配超过所有物理内存和交换空间总和的内存

编辑 `redis.conf` 配置文件（/etc/redis.conf），按需求做出适当调整，比如：

	#转为守护进程，否则启动时会每隔5秒输出一行监控信息
	daemonize yes 
	#减小改变次数，其实这个可以根据情况进行指定
	save 60 1000 
	#分配256M内存
	maxmemory 256000000 

