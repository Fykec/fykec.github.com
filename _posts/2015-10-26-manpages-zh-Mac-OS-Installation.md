# manpages-zh Mac OS Installation

##Compile

	cd <manpages-zh-dir>
	autoreconf -if
	./congfigure --disable-zhtw
	make
	

##Install

###Create new man dir
	cd ~
	mkdir man/zh_CN

###edit profile like .proflie or .zprofile add new man page path

	MANPATH=~/man/zh_CN/:$MANPATH; export MANPATH
	
###Copy

	cd <manpages-zh-dir>
	mkdir UTF-8
	cp -r src/man* UTF-8/
	cp -r UTF-8/* ~/man/zh_CN/
	
##如果显示中文出现乱码 检查 groff 版本, MAC 上的的groff 版本为1.1x 不够新，最新的1.22.x了

	brew install groff

### 修改man的配置文件 sudo emacs /private/etc/man.conf， 修改NROFF为：
	
	NROFF           preconv -e utf8 | /usr/local/bin/groff -Wall -mtty-char -Tutf8 -mandoc -c 		
		
##Reference
1. [Linux 打造 man 中文帮助手册图解](http://blog.jobbole.com/93406/)
2. [MAC 系统中显示中文MAN手册](https://linux.cn/article-2163-1.html)