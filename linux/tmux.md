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

* 正确安装后`/usr/local/lib`目录中会生成若干文件

		lrwxrwxrwx. 1 root root      21 Sep  7 16:37 libevent-2.0.so.5 -> libevent-2.0.so.5.1.9
		-rwxr-xr-x. 1 root root  968682 Sep  7 16:37 libevent-2.0.so.5.1.9
		-rw-r--r--. 1 root root 1571682 Sep  7 16:37 libevent.a
		lrwxrwxrwx. 1 root root      26 Sep  7 16:37 libevent_core-2.0.so.5 -> libevent_core-2.0.so.5.1.9
		-rwxr-xr-x. 1 root root  585225 Sep  7 16:37 libevent_core-2.0.so.5.1.9
		-rw-r--r--. 1 root root  978386 Sep  7 16:37 libevent_core.a
		-rwxr-xr-x. 1 root root     976 Sep  7 16:37 libevent_core.la
		lrwxrwxrwx. 1 root root      26 Sep  7 16:37 libevent_core.so -> libevent_core-2.0.so.5.1.9
		lrwxrwxrwx. 1 root root      27 Sep  7 16:37 libevent_extra-2.0.so.5 -> libevent_extra-2.0.so.5.1.9
		-rwxr-xr-x. 1 root root  404852 Sep  7 16:37 libevent_extra-2.0.so.5.1.9
		-rw-r--r--. 1 root root  593368 Sep  7 16:37 libevent_extra.a
		-rwxr-xr-x. 1 root root     983 Sep  7 16:37 libevent_extra.la
		lrwxrwxrwx. 1 root root      27 Sep  7 16:37 libevent_extra.so -> libevent_extra-2.0.so.5.1.9
		-rwxr-xr-x. 1 root root     941 Sep  7 16:37 libevent.la
		lrwxrwxrwx. 1 root root      30 Sep  7 16:37 libevent_pthreads-2.0.so.5 -> libevent_pthreads-2.0.so.5.1.9
		-rwxr-xr-x. 1 root root   18430 Sep  7 16:37 libevent_pthreads-2.0.so.5.1.9
		-rw-r--r--. 1 root root   18670 Sep  7 16:37 libevent_pthreads.a
		-rwxr-xr-x. 1 root root    1004 Sep  7 16:37 libevent_pthreads.la
		lrwxrwxrwx. 1 root root      30 Sep  7 16:37 libevent_pthreads.so -> libevent_pthreads-2.0.so.5.1.9
		lrwxrwxrwx. 1 root root      21 Sep  7 16:37 libevent.so -> libevent-2.0.so.5.1.9
		drwxr-xr-x. 2 root root    4096 Sep  7 16:37 pkgconfig


	 
* cd to tmux src
	
		LDFLAGS="-L/usr/local/lib -Wl,-rpath=/usr/local/lib" ./configure --prefix=/usr/local
		make && make install
	 
* you're good to go...




## Troubling Shoot:
### 1. libevent notfound when confiure

* Question: 

> error: "libevent not found"

* Fix:

libevent 编译参数设置错误 `--prefix=/usr/local`


libevent 没有正确安装在 /usr/local/lib 目录下



## Configuration

* 配置文件位于 ~/.tmux.conf  若没有则创建之

		set -g status-fg white
		set -g status-left ‘#[fg=green]#H’
		
		# Reload config
		unbind r
		bind r source-file ~/.tmux.conf \; display "Reloaded!"
		
		# Enable status bar utf8 support
		set -g status-utf8 on
		
		# Set pane
		set -g pane-border-fg green
		set -g pane-border-bg black
		set -g pane-active-border-fg white
		set -g pane-active-border-bg yellow
		
		
		# Set message Highlight
		set -g message-fg white
		set -g message-bg black
		# Set-option -g default-command "reattach-to-user-namespace -l zsh"
		set -g default-terminal "screen-256color"
		set -g history-limit 20000
		
		# Use ctrl-a instead of ctrl-b
		set-option -g prefix C-a
		bind-key C-a last-window
		
		# Rebinding the pane splitting bindings
		unbind % # Remove default binding since we’re replacing
		unbind |
		bind | split-window -h
		unbind _
		bind _ split-window -v
		
		# Set status bar
		set -g status-bg black
		set -g status-fg white
		set -g status-left ‘#[fg=green]#H’
		
		# Reload config
		unbind r
		bind r source-file ~/.tmux.conf \; display "Reloaded!"
		
		# Enable status bar utf8 support
		set -g status-utf8 on
		
		# Set pane
		set -g pane-border-fg green
		set -g pane-border-bg black
		set -g pane-active-border-fg white
		set -g pane-active-border-bg yellow
		
		
		# Set message Highlight
		set -g message-fg white
		set -g message-bg black
		set -g message-attr bright
		
		
		
		# Set status bar
		set -g status-fg white
		set -g status-bg black
		setw -g window-status-fg cyan
		setw -g window-status-bg default
		setw -g window-status-attr dim
		setw -g window-status-current-fg white
		setw -g window-status-current-bg red
		setw -g window-status-current-attr bright
		
		set -g status-left-length 40
		set -g status-left "#[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"
		set -g status-right "#[fg=cyan]%d %b %R"
		set -g status-interval 60
		set -g status-justify centre
		
		
		#Cycle through panes
		unbind ^A
		bind ^A select-pane -t :.+
		
		unbind Right
		bind Right resize-pane -R 8
		unbind Left
		bind Left resize-pane -L 8
		unbind Up
		bind Up resize-pane -U 4
		unbind Down
		bind Down resize-pane -D 4
		
		unbind h
		bind h select-pane -L
		unbind j
		bind j select-pane -D
		unbind k
		bind k select-pane -U
		unbind l
		bind l select-pane -R