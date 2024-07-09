---
title: apache
tags: apache
categories: apache
---

> apache 安装配置

<!--more-->

#### Apache 安装/启动

##### 源码安装

使用国内的清华源，下载Apache相关依赖包

[https://mirrors.tuna.tsinghua.edu.cn/apache//httpd/httpd-2.4.43.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.43.tar.gz)

[https://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-1.6.5.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.6.5.tar.gz)

[https://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-util-1.6.1.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz)

https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz

###### 手动安装

```bash
#相关依赖
[root@localhost ~]#yum -y install gcc gcc-c++ expat-devel

[root@localhost src]#wget https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.43.tar.gz
[root@localhost src]#wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.6.5.tar.gz
[root@localhost src]#wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz
[root@localhost src]#wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz

[root@localhost src]#for tar in *.tar.gz; do tar xvf $tar; done

[root@localhost src]#cp -r apr-1.6.5 /usr/local/src/httpd-2.4.43/srclib/apr
[root@localhost src]#cd apr-1.6.5/
[root@localhost apr-1.6.5]#./configure --prefix=/usr/local/apr
[root@localhost apr-1.6.5]#make && make install

[root@localhost src]#cp -r apr-util-1.6.1 httpd-2.4.43/srclib/apr-util
[root@localhost src]#cd ../apr-util-1.6.1/
[root@localhost apr-util-1.6.1]#./configure --prefix=/usr/local/apr-util \
--with-apr=/usr/local/apr
[root@localhost apr-util-1.6.1]#make && make install

[root@localhost src]#cp -r pcre-8.40 httpd-2.4.43/srclib/pcre
[root@localhost src]#cd ../pcre-8.40/
[root@localhost pcre-8.40]#./configure --prefix=/usr/local/pcre
[root@localhost pcre-8.40]#make && make install

[root@localhost src]#cd httpd-2.4.43
[root@localhost httpd-2.4.43]#./configure \
--prefix=/usr/local/apache2 \
--enable-so \
--enable-proxy \
--enable-proxy-ajp \
--enable-proxy-http \
--with-apr=/usr/local/apr/bin/apr-1-config \
--with-apr-util=/usr/local/apr-util/bin/apu-1-config \
--with-pcre=/usr/local/pcre
[root@localhost httpd-2.4.43]#make && make install

```

###### 脚本安装

```bash
#!/bin/bash

DIR=$(cd $(dirname $0);pwd)
[ ! -d $DIR/log ] && mkdir -p $DIR/log

function install_tar() {
	FILENAME=$(basename $1)
	echo "build & install $FILENAME"

	{
		cd $DIR

		if [ -d "$2" ]; then
			rm -rf "$2"
		fi

		case "$1" in
			*.tar.gz|*.tgz) tar xzvf $1;;
			*.tar.bz2) tar xjvf $1;;
			*) tar xvf $1;;
		esac

		#cp -r $2 httpd-2.4.43/srclib/$2

		cd "$2"

		case "$3" in
			apr)
				./configure --prefix=/usr/local/apr
				make
				make install
				rm -Rf $DIR/$2
				;;
			apr-util)
				./configure --prefix=/usr/local/apr-util \
				--with-apr=/usr/local/apr
				make
				make install
				rm -Rf $DIR/$2
				;;
				
            		pcre)
				./configure --prefix=/usr/local/pcre
				make
				make install
				rm -Rf $DIR/$2
				;;
			httpd)
				./configure \
				--prefix=/usr/local/apache2 \
				--enable-so \
				--enable-proxy \
				--enable-proxy-ajp \
				--enable-proxy-http \
				--with-apr=/usr/local/apr/bin/apr-1-config \
				--with-apr-util=/usr/local/apr-util/bin/apu-1-config \
				--with-pcre=/usr/local/pcre
				make
				make install
				rm -Rf $DIR/$2
				;;
		esac
	} &> $DIR/log/install-$FILENAME.log
}

{
	wget -P $DIR https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.43.tar.gz
	wget -P $DIR https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz
	wget -P $DIR https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz
	wget -P $DIR https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.6.5.tar.gz
} &> $DIR/log/wget.log

yum -y install gcc gcc-c++ expat-devel  > $DIR/log/yum.log

install_tar apr-1.6.5.tar.gz apr-1.6.5 apr
install_tar apr-util-1.6.1.tar.gz apr-util-1.6.1 apr-util
install_tar pcre-8.40.tar.gz pcre-8.40 pcre
install_tar httpd-2.4.43.tar.gz httpd-2.4.43 httpd

echo finish.

```

###### 启动Apache

```bash
[root@localhost bin]# ll /usr/local/apache2/bin
total 2804
-rwxr-xr-x. 1 root root  100440 Jun 22 14:16 ab
-rwxr-xr-x. 1 root dip     3437 Jun 22 14:15 apachectl
-rwxr-xr-x. 1 root dip    23879 Jun 22 14:15 apxs
-rwxr-xr-x. 1 root root   13219 Jun 22 14:16 checkgid
-rwxr-xr-x. 1 root dip     8925 Jun 22 14:15 dbmmanage
-rw-r--r--. 1 root dip     1073 Jun 22 14:15 envvars
-rw-r--r--. 1 root dip     1073 Jun 22 14:15 envvars-std
-rwxr-xr-x. 1 root root   23961 Jun 22 14:16 fcgistarter
-rwxr-xr-x. 1 root root   79919 Jun 22 14:16 htcacheclean
-rwxr-xr-x. 1 root root   47484 Jun 22 14:16 htdbm
-rwxr-xr-x. 1 root root   25313 Jun 22 14:16 htdigest
-rwxr-xr-x. 1 root root   49260 Jun 22 14:16 htpasswd
-rwxr-xr-x. 1 root root 2365099 Jun 22 14:16 httpd
-rwxr-xr-x. 1 root root   22102 Jun 22 14:16 httxt2dbm
-rwxr-xr-x. 1 root root   25222 Jun 22 14:16 logresolve
-rwxr-xr-x. 1 root root   43645 Jun 22 14:16 rotatelogs

[root@localhost bin]# ./apachectl start
httpd: Could not reliably determine the server's fully qualified domain name, using xxx for ServerName
[root@localhost bin]# ./apachectl stop
httpd: Could not reliably determine the server's fully qualified domain name, using xxx for ServerName


去除启动或停止时“server's fully qualified domain name提示”：
在/usr/local/apache2/conf/httpd.conf配置文件中搜索“ServerName”关键字，
去掉前面的“#”并修改后面的主机名即可。
如：ServerName test

修改后重启服务发现没有相关提示。
[root@localhost bin]# ./apachectl stop
[root@localhost bin]# ./apachectl start
```

###### 设置开机自启动

1）编写apache启动脚本

```bash
[root@localhost script]# pwd
/opt/script
[root@localhost script]#cat httpd
#!/bin/bash
#chkconfig: 345 85 15
#description:Start and stop the Apache HTTP Server

dir="/usr/local/apache2/bin"

function httpd_start(){
        $dir/apachectl start

}

function httpd_stop(){
        $dir/apachectl stop

}

case $1 in
        start)
                httpd_start
                ;;

        stop)
                httpd_stop
                ;;

        restart)
                httpd_stop
                httpd_start
                ;;

        *)
                echo "Usage: httpd start|stop|restart"
                ;;
esac
```

 注意：以下两行内容必须写，并且要写在第二行，否则无法设置开机自启动

```bash
#chkconfig:345 85 15
#description:Start and stop the Apache HTTP Server
```

  2）把编写的启动脚本复制到/etc/init.d/目录下，并且赋予执行权限，（添加到系统服务中）

```bash
[root@localhost script]# chmod +x httpd 
[root@localhost script]# ll
total 4
-rwxr-xr-x. 1 root root   386 Jun 22 17:41 httpd
[root@localhost script]# mv httpd /etc/init.d/
```

  3）重新加载守护进程，启动服务

```bash
[root@localhost script]# systemctl daemon-reload   #必须先加载守护进程，否则无法启动服务
[root@localhost script]# systemctl start httpd
[root@localhost script]# netstat -lnpt|grep httpd
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2996/httpd
```

  注意：将编写的apache启动脚本添加到系统服务中后，首先要加载守护进程，否则无法启动服务。

  下面是没有加载守护进程，启动服务的报错信息：

```bash
[root@localhost script]# systemctl start httpd
Warning: httpd.service changed on disk. Run 'systemctl daemon-reload' to reload units.
```

  4）设置开机自启动

​    设置开机自启动命令：

```bash
[root@lamp scripts]# chkconfig --add httpd
```

​    查看开机自启动是否设置成功：

```bash
[root@lamp scripts]# chkconfig --list|grep httpd

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

httpd          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

 5）apache安装路径

```bash
[root@localhost apache2]# pwd
/usr/local/apache2/
[root@lamp httpd]# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs  man  manual  modules
```

##### yum 安装



#### Apache 配置文件







#### Apache 共享文件

1. 软连接方式
2. 别名方式



#### Apache 访问控制

1. 开启基本认证
   - 单人访问
   - 多人访问
2. 网络访问控制





#### Apache 虚拟主机

1. 基于IP的虚拟主机

2. 基于端口的虚拟主机

3. 基于域名的虚拟主机

   

#### 常用状态码分类：

200：成功，请求的所有的数据通过响应报文的entity-body部分发送；ok
301：请求的URL指向的资源已经被删除，但是在响应报文中通过首部Location指明了资源现在所处的新位置；Moved Permanently
302：与301相似，但是在响应报文中通过Location指明资源现在所处的临时的新位置；Found
304：客户端发出了条件式请求，但是服务器上的资源未发送改变，通过响应此响应状态码通知客户端，Not Modified
401：需要客户端输入账号和密码才能访问资源；Unauthorized
403：请求被禁止；Forbidden
404：服务器无法找到客户端请求的资源；Not Found
500：服务器内部错误；Internal Server Error
502：代理服务器从后端服务器收到了一条伪响应;Bad Gateway