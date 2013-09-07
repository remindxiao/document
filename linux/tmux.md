Install Guide
===============

* download latest libevent2 and tmux sources, and extract them somewhere

* at the time of writing:
	
		wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
	
		wget http://downloads.sourceforge.net/tmux/tmux-1.7.tar.gz
	 
	 
	


* install deps
	
		yum install gcc kernel-devel make ncurses-devel
		 
* cd to libevent2 src

		./configure --prefix=/usr/local
		make && make install
	 
* cd to tmux src
	
		LDFLAGS="-L/usr/local/lib -Wl,-rpath=/usr/local/lib" ./configure --prefix=/usr/local
		make && make install
	 
* you're good to go...