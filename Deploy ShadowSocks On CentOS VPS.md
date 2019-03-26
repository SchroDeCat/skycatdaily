---
layout: '[page]'
title: Deploy ShadowSocks On CentOS 7 Server
date: 2018-12-10 5:53:23
tags:
---
# 缘起
平时都会用到google用来学习，facebook也玩玩，所以你懂得。现在是非常时期，许多VPN商都倒了。只能自己搭建。我用的主机是Vultr的，配置如下：CPU:1 vCore，RAM:1024 MB，Storage:25 GB SSD，每月5刀，刚开始用的是东京节点，但是容易掉包。后来切换到新加坡节点。
Vultr现在支持支付宝了，很方便。大家如果购买请使用我的推广链接，https://www.vultr.com/?ref=7159825

# Shadowsocks
Shadowsocks(ss) 是由 Clowwindy 开发的一款软件，其作用本来是加密传输资料。当然，也正因为它加密传输资料的特性，使得 GFW 没法将由它传输的资料和其他普通资料区分开来，也就不能干扰我们访问那些「不存在」的网站了。

# 安装
我的服务器是Centos7内核如下：

	[root@rootrl var]# uname -a
	Linux rootrl 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux

本次使用python版本的shadowsocks，其他版本还有如go这样较为方便部署的类型。
## 0.配置SSH密钥访问
为了方便配置服务器
首先将本地的共钥写入

	~/.ssh/authorized_keys
	
然后配置

	/etc/ssh/sshd_config

使用vi工具进行编辑，确认或添加：

	PermitRootLogin yes
	RSAAUthentication yes
	PubkeyAuthentication yes
	AuthorizedKeysFile .ssh/authorized_keys

启动ssh的服务

	systemctl start sshd.service
设置开机自动启动ssh服务

	systemctl enable sshd.service
	
设置文件夹~/.ssh的访问权限
	
	chmod 700 ~/.ssh

## 1.下载pip等支持组件：
安装setuptools和pip

	yum install python-setuptools && easy_install pip
安装Python-Gevent,用于提高性能
在终端中执行，命令如下：

	yum install libevent
	yum install python-devel
	pip install gevent

安装M2Crypto，用于加密的第三库
在终端中执行，命令如下：

	yum install openssl-devel
	yum install swig
	pip install M2Crypto
安装Shadowsocks服务端
	
	pip install shadowsocks
	
## 2.配置shadowsocks
按照如下内容使用vi工具编辑 /etc/shadowsocks.json

	{
    "server":"my_server_ip",  
    # Note:Aliyun & Tencent Yun this should be the private IP instead of Public IP
    "server_port":9000,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"rc4-md5"
	}
* server 这里一般填vps的ip地址
* server_port 一般是自定义的，你想用哪个就哪个，建议1000-9999之间
* password 这里是输入密码，修改为自己的密码即可
* method 加密方式推荐使用 rc4-md5，因为 rc4-md5 比 aes-256-cfb速度快好几倍，如果用在路由器上会有显著性能提升

## 3.shadowsocks运行指令
如果需要配置文件在后台运行：

	ssserver -c /etc/shadowsocks.json -d start
	
如果需要停止运行：

	ssserver -c /etc/shadowsocks.json -d stop
	
设置开机启动：

	vi /etc/rc.local

打开文件后加入：

	ssserver -c /etc/shadowsocks.json -d start


# 防火墙
## 对firewalld类防火墙(CentOS 7)
检查防火墙是否允许设定端口进行通信
	
	firewall-cmd --list-service
添加运行端口通讯

	firewall-cmd --permanent --add-port=8989/tcp
之后更新防火墙规则：
	
	firewall-cmd --reload # 更新防火墙规则
使用systemctl控制firewall服务：

	# 安装firewalld
	yum install firewalld firewall-config
	
	systemctl start  firewalld # 启动
	systemctl status firewalld # 或者	firewall-cmd --state 	查看状态
	systemctl disable firewalld # 停止
	systemctl stop firewalld  # 禁用

	# 关闭服务的方法
	# 你也可以关闭目前还不熟悉的FirewallD防火墙，而使用iptables，命令如下：

	systemctl stop firewalld
	systemctl disable firewalld
	yum install iptables-services
	systemctl start iptables
	systemctl enable iptables
	
	# 详情可参考http://wangchujiang.com/linux-command/c/firewall-cmd.html
## 对iptables类防火墙(CentOS 7以前)
检查防火墙是否允许你设定的端口进行通信

	iptables -L -n | grep 8989
如果没有信息的话，就是防火墙不允许该端口进行通信。
需设置：

	iptables -I INPUT -p tcp --dport 8989 -j ACCEPT
# 加速
开启TCP Fast Open
	
	vim /etc/rc.local
在最后一行增加以下内容
	
	echo 3 > /proc/sys/net/ipv4/tcp_fastopen
然后

	vim /etc/sysctl.conf
在最后一行增加：

	net.ipv4.tcp_fastopen = 3
编辑配置文件
	
	vim /etc/shadowsocks/config.json
添加一项

	"fast_open":true
最后重启
	
	/etc/init.d/shadowsocks restart
# 软件加速
加速有锐速加速和Google BBR加速。我这里使用的是BBR加速

我这里参考的：https://teddysun.com/489.html

## 安装一键脚本:

	wget --no-check-certificate https://github.com/	teddysun/across/raw/master/bbr.sh
	chmod +x bbr.sh
	./bbr.sh
安装按成后会提示重启，重启完成后：
# 查看内核：
	uname -r
包含4.13就说明内核替换成功

	#检查是否开启BBR
	sysctl net.ipv4.tcp_available_congestion_control
	#返回值一般为
	net.ipv4.tcp_available_congestion_control = bbr cubic reno
	sysctl net.ipv4.tcp_congestion_control
	# 返回值一般为：
	net.ipv4.tcp_congestion_control = bbr
	sysctl net.core.default_qdisc
	# 返回值一般为：
	net.core.default_qdisc = fq
	lsmod | grep bbr
	# 返回值有tcp_bbr则说明已经启动
客户端:
到https://github.com/shadowsocks 下载你需要的客户端

# 转载
https://cloud.tencent.com/developer/article/1159683
https://rootrl.github.io/2017/10/11/Vultr-centos%E5%AE%89%E8%A3%85shadowsocks%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%B9%B6%E5%BC%80%E5%90%AFBBR%E5%8A%A0%E9%80%9F/
https://github.com/shadowsocks/shadowsocks/issues/298

