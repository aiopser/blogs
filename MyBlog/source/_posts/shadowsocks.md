---
title: shadowsocks
tags: linux
categories: linux
---

> shadowsocks 安装配置

<!--more-->

转载：https://crifan.github.io/scientific_network_summary/website/server_client_mode/ss_server/self_build_ss_server.html

本文以CentOS7为例，介绍如何搭建自己的ss服务器。

#### 准备工作

在自己搭建ss服务器之前，首先要求你自己有一台可以访问国外网站的服务器（比如[Linode](https://www.linode.com/)、[Vultr](https://www.vultr.com/)、[DigitalOcean](https://www.digitalocean.com/)等）主机提供商购买一个VPS主机，这样可以远程通过SSH工具登录主机，然后再去安装ss服务。

我使用的是阿里云新加坡的轻量级服务器 [点击链接](https://common-buy.aliyun.com/?commodityCode=swas&regionId=ap-southeast-1#/buy)  然后我们在此服务器上安装shadowsocks

#### 安装方法一

用Python的shadowsocks实现ss服务

```python
#系统默认是已经安装了python, 查看下版本号
[root@aliyun ~]# python --version
Python 2.7.5

#Python包管理工具：pip 默认已安装，如果没有安装，执行下面的命令完成安装
[root@aliyun ~]#yum -y install python-setuptools 
[root@aliyun ~]#easy_install pip
[root@aliyun ~]#pip install --upgrade pip

#pip安装shadowsocks
[root@aliyun ~]#pip install shadowsocks

#手动添加配置文件/etc/shadowsocks.json
#内容可以参考下面的安装方法二中的shadowsocks-libev的配置文件/etc/shadowsocks-libev/config.json
[root@aliyun ~]# cat >/etc/shadowsocks.json<<-EOF
{
  "server": "0.0.0.0",
  "server_port": 21198,
  "password": "ywt123!",
  "method": "aes-256-cfb",
  "timeout": 300,
  "mode": "tcp_and_udp"
}
EOF

```

##### 启动ss
```
[root@aliyun ~]#ssserver -c /etc/shadowsocks.json -d start
```

##### 停止ss
```
[root@aliyun ~]#ssserver -c /etc/shadowsocks.json -d stop
```

##### 查看ss服务状态

```
ps -ef |grep ssserver
```



#### 安装方法二

用shadowsocks-libev实现ss服务

```python
[root@aliyun ~]#wget -P /etc/yum.repos.d \
https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
[root@aliyun ~]#yum install -y shadowsocks-libev

#安装完成后默认会生成配置文件
/etc/shadowsocks-libev/config.json

配置参数介绍：
server				当前自己的服务器的地址。比如：0.0.0.0
server_port			ss服务的端口。一般要大于1024，小于65536，可以随意取值，只要不和其他端口冲突即可
password			ss客户端使用ss时要使用的密码。
method				加密方式。以前常见的方式是：aes-256-cfb
				最新更加复杂但更安全的是：chacha20-ietf-poly1305
    				注意需要ss客户端要支持该加密方式才能正常使用ss服务
timeout				超时时间，单位：秒
mode				ss服务的模式。比如：tcp_and_udp，即支持tcp也支持udp

```

##### 启动ss

```
systemctl start shadowsocks-libev
```

设置开机启动：

```
systemctl enable shadowsocks-libev
```

##### 停止ss

```
systemctl stop shadowsocks-libev
```

重新启动

```
systemctl restart shadowsocks-libev
```

##### 查看ss服务状态：

```
systemctl status shadowsocks-libev
```

#### 脚本安装

```bash
[root@aliyun ~]#cat ss.sh
#!/bin/bash
pip -V >/dev/null 2>&1
if [[ $? -ne 0 ]];then
	echo "error,Not installed" 
	yum -y install python-setuptools
	easy_install pip
	pip install --upgrade pip
else
	echo "pip already installed"
fi
# install ss
pip install shadowsocks
# configure ss file
cat >/etc/shadowsocks.json<<-EOF
{
  "server": "0.0.0.0",
  "server_port": 21198,
  "password": "ywt123!",
  "method": "aes-256-cfb",
  "timeout": 300,
  "mode": "tcp_and_udp"
}
EOF
# start ss
ssserver -c /etc/shadowsocks.json -d start

```

