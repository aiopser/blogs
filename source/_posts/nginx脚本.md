---
title: nginx 启动脚本
tags: nginx
categories: nginx
---
>nginx 启动脚本

<!--more-->
# nginx 开机自启动方法

## 自定义脚本

在linux系统的/etc/init.d/目录下创建nginx文件，使用如下命令：

```
vim /etc/init.d/nginx
```

在脚本中添加如下命令：此脚本来源nginx官方示例：https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/

```bash
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
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
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
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
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

如果是自定义编译安装的nginx，需要根据您的安装路径修改下面这两项配置：

- nginx=”/usr/sbin/nginx” 修改成nginx执行程序的路径。

- NGINX_CONF_FILE=”/etc/nginx/nginx.conf” 修改成配置文件的路径。

保存脚本文件后设置文件的执行权限：

```
chmod a+x /etc/init.d/nginx
然后，就可以通过该脚本对nginx服务进行管理了：
/etc/init.d/nginx start
/etc/init.d/nginx stop
```

使用chkconfig进行管理

上面的方法完成了用脚本管理nginx服务的功能，但是还是不太方便，比如要设置nginx开机启动等。这时可以使用chkconfig来设置。

先将nginx服务加入chkconfig管理列表：

```
chkconfig --add /etc/init.d/nginx
加完这个之后，就可以使用service对nginx进行启动，重启等操作了。
service nginx start
service nginx stop
```

设置终端模式开机自启动：

```
chkconfig nginx on
```

## systemctl启动

在/usr/lib/systemd/system 创建nginx.service文件

```
vim /usr/lib/systemd/system/nginx.service
```

文件内容如下：

```bash
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginxsbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

文件内容解释

```bash
[Unit]			#服务的说明
Description		#描述服务
After			#描述服务类别

[Service]		#服务运行参数的设置
Type=forking	#是后台运行的形式
ExecStart		#服务的具体运行命令
ExecReload		#重启命令
ExecStop		#停止命令
PrivateTmp=True	#表示给服务分配独立的临时空间

#注意：启动、重启、停止命令全部要求使用绝对路径

[Install]		#服务安装的相关设置，可设置为多用户
```

常用命令

```bash
systemctl daemon-reload				#使配置文件生效
systemctl start nginx				#启动nginx
systemctl stop nginx				#停止nginx
systemctl restart nginx				#重启nginx
systemctl enable nginx				#设置开机自启动nginx
systemctl disable nginx				#停止开机自启动
systemctl is-enabled nginx			#查询nginx是否开机启动
systemctl list-units --type=service #查看所有已启动的服务

```

## nginx相关报错

**报错信息：**

Failed to read PID from file /run/nginx.pid: Invalid argument

**问题产生原因**：

因为 nginx 启动需要一点点时间，而 systemd 在 nginx 完成启动前就去读取 pid file
造成读取 pid 失败

**解决方法**：

让 systemd 在执行 ExecStart 的指令后等待一点点时间即可
如果你的 nginx 启动需要时间更长，可以把 sleep 时间改长一点
建立目录

```
mkdir -p /etc/systemd/system/nginx.service.d
```

在新建目录中建立文件override.conf，输入内容

```
[Service]
ExecStartPost=/bin/sleep 0.1
```

然后

```
systemctl daemon-reload
systemctl restart nginx.service
```

