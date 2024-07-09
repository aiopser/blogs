---
tiele: 两台服务器搭建nginx负载
tags: nginx
categories: nginx
---

> 基于两台服务的nginx负载均衡

<!--more-->

#### 准备工作

服务器1: 192.168.2.101

服务器2: 192.168.2.102

配置域名到服务器1: 192.168.2.101

本地测试 直接在Windows hosts文件里 添加域名解析

路径是：C:\Windows\System32\drivers\etc，打开hosts文件添加ip，域名

192. 168.2.101	www.upstream.com

#### nginx配置

服务器1: `192.168.2.101`  nginx.conf配置

```
upstream  www{
     server 192.168.2.101:81;
     server 192.168.2.102:81;
}



server {
        listen       80;
        server_name  www.upstream.com;
       
        location / {
                 proxy_pass http://www/;
                 proxy_set_header Host   $host;
                 proxy_connect_timeout 2s;


        }


        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

在服务器1: `192.168.2.101` 和 服务器2: `192.168.2.102` 中继续加入配置（配置你的项目访问地址）

```
server {
        listen       81;
        server_name  localhost;
        root         /usr/share/nginx/html/web81;


        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
        location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
    }
```

```
服务器1: 192.168.2.101
cat /usr/share/nginx/html/web81/index.html
server1

服务器2: 192.168.2.102
cat /usr/share/nginx/html/web81/index.html
server2


```

访问 http://www.upstream.com/  出现server1 server2 表示成功



#### nginx 完整配置

**服务器1: 192.168.2.101**

[root@ywt etc]#tree nginx
nginx
├── conf.d
│   ├── default.conf
│   └── server81.conf
├── default.d
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── modules -> ../../usr/lib64/nginx/modules
├── nginx.conf
├── nginx.conf.rpmsave
├── scgi_params
├── uwsgi_params
└── win-utf



```bash
[root@ywt nginx]# cat nginx.conf

# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    server_tokens       off;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
upstream www{
    server 192.168.2.101:81;
    server 192.168.2.102:81;

}

#===============我是分割线=======================================================

[root@ywt nginx]# cat conf.d/default.conf
server {
        listen       80 default_server;
        #server_name  www.tengxunyun.com;
        server_name  localhost;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        # include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
#===============我是分割线====================================================

[root@ywt nginx]# cat conf.d/server81.conf 
server {
        listen       81;
        server_name  localhost;
        root         /usr/share/nginx/html/web81;

        # Load configuration files for the default server block.
        # include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
	location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
	}
    }


```



服务器2: 192.168.2.102 nginx配置文件如下：

[root@ywt etc]#tree nginx
nginx
├── conf.d
│   ├── default.conf
│   └── server81.conf
├── default.d
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── modules -> ../../usr/lib64/nginx/modules
├── nginx.conf
├── nginx.conf.rpmsave
├── scgi_params
├── uwsgi_params
└── win-utf

```bash
[root@ywt nginx]# cat nginx.conf

# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    server_tokens       off;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

#===============我是分割线=====================================================

[root@ywt nginx]# cat conf.d/default.conf
server {
        listen       80 default_server;
        server_name  localhost;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        # include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
#===============我是分割线======================================================

[root@ywt nginx]# cat conf.d/server81.conf 
server {
        listen       81;
        server_name  localhost;
        root         /usr/share/nginx/html/web81;

        # Load configuration files for the default server block.
        # include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
	location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
	}
    }


```

