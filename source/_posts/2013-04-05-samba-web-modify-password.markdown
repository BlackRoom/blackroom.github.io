---
layout: post
title: "让Samba用户通过WEB页面修改密码"
date: 2013-04-05 17:20
comments: true
categories: 
---

前面一篇文章介绍了Samba文件共享服务器的安装及配置方法，最近遇到一个问题所有的用户密码只能通过登录到linux系统修改，而大多数用户是不允许登录到linux系统服务器的。很多其他部门的同事作为Windows终端的用户根本没听说过Linux，更别说使用了。
<!-- more -->
也考虑过把samba建成pdc，windows加入域中，samba用户通过windows登陆域，samba用户在windows上按“ctrl+alt+del”修改密码，或是采用LDAP来做。

后来找到一种更简单的方法使用changepassword这个软件让用户通过web修改改密码，这种看似是比较简单且人性化的了。

##一.简介
changepassword是能让用户从web界面修改系统密码的一款软件，它并没有让你获得修改samba密码的功能。  
而如何实现修改samba密码的功能呢，就要用到pam_smbpass.so这个模块，它才是真正能让samba密码和系统密码同步的利器。  

实际上的原理其实是，用户通过Web页面使用changepassword来修改系统的密码，然后由pam_smbpass.so模块自动将系统密码同步给了samba，以达到用户修改samba密码的目的。
 
##二.安装配置Changerpassword

系统环境：
操作系统：CentOS 6.4
已装服务：Samba，httpd 且启动正常

####1.安装changerpassword以实现用户利用web页面修改密码

从changerpassword的官网下载[changepassword](http://changepassword.sourceforge.net/)

	# wget http://sourceforge.net/projects/changepassword/files/changepassword/0.9/changepassword-0.9.tar.gz
	# yum -y install gcc
	#tar zxvf changepassword-0.9.tar.gz
	#cd changepassword-0.9
 
修改**conf.h**头文件，设置软件修改密码使用的临时目录（本来为/tmp，但是实际上是不能用的，要新建一个权限为777的目录）  
将前三行的定义修改为自己创建的目录（这里我修改到了/var/smbchangepwd目录下）：

	# vim conf.h
	// temporary directory and files to use
	char TMPFILE[]=”/var/smbchangepwd/changepassword-shadow-XXXXXX”;
	char TMPSMBFILE[]=”/var/smbchangepwd/changepassword-smb-XXXXXX”;
	char TMPSQUIDFILE[]=”/var/smbchangepwd/changepassword-squid-XXXXXX”;

创建需要用到的目录（第二个为编译configure时候用到的cgidir）

	mkdir –pv /var/smbchangepwd
	mkdir –pv /home/admin/www/samba-change-passwd

编译安装

	# ./configure –enable-cgidir=/home/admin/www/samba-change-passwd –enable-language=English –enable-smbpasswd=/etc/samba/smbpasswd –disable-squidpasswd –enable-logo=opentech.jpg
 
这里解释一下:

	–enable-cgidir	这个目录是Web页面要读取的目录，一般可以设置为网站的根目录，或者网站根目录下的某个目录，比如/var/www/smb/，程序会将最后的web访问页放在这个目录中。
	–enable-language	设置程序的显示语言，里面支持Chinese
	–enable-smbpasswd	smb的密码文件存放位置
	–disable-squidpasswd	禁用squid同步密码
	–enable-logo		这是装饰Web页面中的标题的图片，可以随便指定，只要是http支持的图片格式都可以，需要我们手动放一个图片在cgidir中。

按照官方的来的话这里只要直接make，完后make install 即可，但是，从我自己安装的经验来看，这里一定会报错的，报错如下：

	DSMBPASSWD=\”/etc/samba/smbpasswd\” -DSQUIDPASSWD=\”no\” -DLOGO=\”none\” -L./smbencrypt –ldes
	/usr/bin/ld: skipping incompatible ./smbencrypt/libdes.a when searching for –ldes
	/usr/bin/ld: cannot find –ldes
	collect2: ld returned 1 exit status
	make: *** [changepassword.cgi] Error 1

从报错可以看到/usr/bin/ld: cannot find –ldes ，网上有不少解决办法，实际上那都无法解决根本问题，而官方实际上也知道会遇到这个问题，于是我们只需重新编译加载libdes即可

	# cd smbencrypt/
	# tar -xzvf libdes-4.04b.tar.gz
	# cd des/
	# make
	# cp libdes.a ../
	# cd ../..

这时从新make,make install即可完成安装。

	# make
	# make install

安装程序会拷贝一个叫changepassword.cgi的文件到我们指定的–cgidir目录，这时，只要我们配置好http,确保能从web直接访问到这个文件即可。当然，别忘了拷贝一个你喜欢的图片到–cgidir所指定的那个目录,名字当然就用那个–logo的名字~
这个根据自己Web配置不同自己添加。

####2.配置Apache

设置apache支持cgi模，搜索cgi 去掉如下注释

	# vi /usr/local/apache2/conf/httpd.conf
	--------------
	LoadModule cgid_module modules/mod_cgid.so
	AddHandler cgi-script .cgi
	--------------

	搜索 DocumentRoot,在/usr/local/apache2/htdocs类目下找到Options选项，修改为.

	--------------
	Options Indexes FollowSymLinks ExecCGI
	--------------

重启服务

	# /usr/local/apache2/bin/apachectl restart

OK,一切就绪后，打开Web，在浏览器中输入：

	http://ip地址/如果还有目录/changepassword.cgi

####3.常见错误提示：

#####a.提示无法更改临时目录，解决方法：查看/var/smbchangepwd临时目录权限是否为777 还要要在编译前把目录改为777。  
#####b.提示没有的用户或用户无效，在下面添加完pam模块后请务必注释掉默认的passdb backend = tdbsam项，改为passdb backend = smbpasswd    确认smbpasswd -a 添加的用户在 /etc/samba/smbpasswd文件中
 
至此已经配置好web页面改密码但是现在改的只是系统的密码还没有配置samba同步系统密码。

##三.实现samba与系统密码同步

实际上配置samba与系统密码同步的原理十分简单，我们都知道密码都是由Pam进行管理的，理论上，当我们使用命令来修改系统密码的时候是调用了pam的密码管理机制，才修改成功的，那么我们其实只要在Pam里加上当修改系统密码的时候也一起让pam把samba的密码给修改掉，就可以了。

samba官方提供的专门用于使用pam来管理密码的模块：pam_smbpass.so  
它的位置位于：

	x86 : /lib/security/pam_smbpass.so
	x64 : /lib64/security/pam_smbpass.so

然后我们只需要将这个模块加入到密码验证的机制里即可，编辑**system-auth**这个pam文件，修改里面的password段插入一行新的password行（这里我的system-auth的配置，注意我加了一行关于pam_smbpass.so的内容）

	# vim /etc/pam.d/system-auth
	auth        required      pam_env.so
	auth        sufficient    pam_fprintd.so
	auth        sufficient    pam_unix.so nullok try_first_pass
	auth        requisite     pam_succeed_if.so uid >= 500 quiet
	auth        required      pam_deny.so

	account     required      pam_unix.so
	account     sufficient    pam_localuser.so
	account     sufficient    pam_succeed_if.so uid < 500 quiet
	account     required      pam_permit.so

	password    requisite     pam_cracklib.so try_first_pass retry=3 type=
	password    requisite     pam_smbpass.so nullok use_authtok try_first_pass
	password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
	password    required      pam_deny.so

	session     optional      pam_keyinit.so revoke
	session     required      pam_limits.so
	session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
	session     required      pam_unix.so

然后保存，这时理论上，当你修改系统密码的时候，关联的这个模块也会修改samba的密码，但是这还不够，我们还要对samba进行一些设置，在**[global]**段设置samba的加密方式为

	# vim /etc/samba/smb.conf
        security = user
        passdb backend = smbpasswd
        encrypt passwords = yes
        smb passwd file = /etc/samba/smbpasswd
        pam password change = yes

注意，请务必注释掉默认的**passdb backend = tdbsam**项改为**passdb backend = smbpasswd**要不然利用**smbpasswd -a** 添加用户不会添加到指定的文件中/etc/samba/smbpasswd  
还有一点需要注意添加完pam_smbpass.so 这个模块后所有的用户必须使用smbpasswd -a 添加到samba中要不然系统用户改不了密码  

然后重启samba:

	# /etc/init.d/smb restart

如果一切正确的话，在/etc/samba下应该已经有一个 smbpasswd这个文件了。这个文件里记录的就是所有可以登陆samba的用户以及密码，初始情况下应该是空才对。

接下来就需要我们手动使用**smbpasswd –a** 往里添加用户了。  
注意：只有在smbpasswd中已经存在的系统用户，当你修改该系统用户的密码的时候，才会一同修改smbpasswd中的用户。这样，我们就达成了让用户从Web修改自己用户系统密码，然后同步到smb的任务。
 
将系统所有用户导入到samba  （注意此时导入的samba用户好像密码无法登陆需在添加完pam模块后更改下系统用户的密码方可）

	cat /etc/passwd |mksmbpasswd.sh > /etc/samba/smbpasswd
	smbpasswd +用户   改密samba密码
	echo passwd123 |passwd --stdin username    将username用户的密码直接改为passwd123


[Samba文件服务器配置](http://blog.blackroom.cn/blog/2013/03/28/smba-wen-jian-fu-wu-qi/)
