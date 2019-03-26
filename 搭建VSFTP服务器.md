---
layout: '[page]'
title: Establish FTP Server
tags:
---
## 1.关于VSFTP

在 Linux 下搭建 FTP 服务器一般会使用 vsftpd。vsftpd 的前两个字母代表 "very secure" 。
参考资料参阅[vsftp官网](https://security.appspot.com/vsftpd.html).

## 2.安装VSFTP
1. 使用yum安装vsftp

		sudo yum install vsftpd
	
2. 如果需要连接其他ftp服务器，则可以安装ftp客户端

		sudo yum install ftp


## 3.添加用户
1.	添加用户并设置密码

		adduser userftp
		passwd userftp

2. 禁止用户的ssh登录权限，只允许ftp访问

		usermod -s /sbin/nologin userftp

## 4.配置VSFTP
1. 打开配置文件
		```bash
		sudo vi /etc/vsftpd/vsftpd.conf
		```
2. 关闭匿名访问

		anonumous_enbale=NO
3. 去掉local_enable的注释，修改为开启，允许本地用户访问ftp文件夹

		local_enable=YES
4. 限制用户仅能访问自己的主目录

		chroot_local_user=YES
		
当我们限定了用户不能跳出其主目录之后，使用该用户登录FTP时往往会遇到这个错误:
		
		500 OOPS: vsftpd: refusing to run with writable root inside chroot () 
		
从2.3.5之后，vsftpd增强了安全检查，如果用户被限定在了其主目录下，则该用户的主目录不能再具有写权限了！如果检查发现还有写权限，就会报该错误。

	 Add stronger checks for the configuration error of running with a writeable root directory inside a chroot(). This may bite people who carelessly turned on chroot_local_user but such is life.  

要修复这个错误，可以用命令chmod a-w /home/user去除用户主目录的写权限，注意把目录替换成你自己的。或者你可以在vsftpd的配置文件中增加下列两项中的一项：


	allow_writeable_chroot=YES
	
5. 设置用户的主目录：（不设置时，默认为用户的家目录**\home\userftp**）

		local_root=/data/test
6. 重启服务

		systemctl restart vsftpd.service
7. 设置开启自启动

	chkconfgi vsftpd on



## 5.连接测试
ftp userftp@111.230.172.245

## 6.卸载VSFTP
1.	查找vsftpd

	```bash		
	rpm -aq vsftpd
	```
2. 卸载vsftpd

		rpm -e 以上查找结果
3.	使用stop、start操作验证删除

		/sbin/service vsftpd stop
		/sbin/service vsftpd start

## 7.参考
1.	[Digital Ocean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-on-centos-6--2)
2. [51CTO](http://koulitsu.blog.51cto.com/7355117/1221441)
3. [Original Artical](http://koulitsu.blog.51cto.com/7355117/1221441)

