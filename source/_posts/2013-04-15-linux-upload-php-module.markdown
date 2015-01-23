---
layout: post
title: "linux下加载PHP扩展模块的一种方法"
date: 2013-03-28 16:14
comments: true
categories: 
---

linux下加载PHP扩展模块程序的一种方法！！ 

我们在使用php源码包安装时，加载php的扩展模块非常方便，使用confgiure命令配置时加载相关模块非常的灵活。但是如果是rpm包安装的该如何自如的加载模块呢？

php提供了一个开发包**php-devel**的RPM包，安装后，会在系统/usr/bin目录下新增了两条命令**phpize**和**php-config**。

	phpize	命令是用来准备 PHP 扩展库的编译环境的。
	php-config	主要是负责扩展程序的配置。

**安装方法**

先查看系统中已安装了那些php程序，主要是查看是否安装了php开发包php-devel的rpm包。如果没有，我们需要安装下php-devel 。

	# rpm -qa|grep php
	php-cli-5.3.3-27.el6_5.2.x86_64
	php-mysql-5.3.3-27.el6_5.2.x86_64
	php-common-5.3.3-27.el6_5.2.x86_64
	php-pdo-5.3.3-27.el6_5.2.x86_64
	php-5.3.3-27.el6_5.2.x86_64
	php-xml-5.3.3-27.el6_5.2.x86_64
	php-gd-5.3.3-27.el6_5.2.x86_64
	php-mcrypt-5.3.3-3.el6.x86_64
	php-bcmath-5.3.3-27.el6_5.2.x86_64
	php-mbstring-5.3.3-27.el6_5.2.x86_64

系统中并没有安装php-devel的rpm包，需要安装一下，可以去官网下载rpm包，也可以使用yum安装非常方便。

	# yum -y install php-devel


安装成功后，会在系统/usr/bin目录下新增了两条命令**phpize**和**php-config**。

**使用方法**

以安装php protobuf模块为例，其他模块雷同。
	
	# git clone https://github.com/allegro/php-protobuf.git
	# cd php-protobuf
	# phpize
	# ./configure
	# make
	# make test
	# make install

下载软件包，我这里软件包在github上所以直接用git克隆下来，进入软件包执行 **phpize** 准备PHP扩展库的编译环境，后面就是编译安装了。

安装成功后，我们需要修改php.ini配置文件.

	# echo "extension=protobuf.so" > /etc/php.d/protobuf.ini
	# php -m

可以使用 **php -m** 命令查看下是否加载了protobuf模块。  
也可以使用简单的php网页查看php的安装信息。

	<?
	phpinfo()
	?>

