---
title: Tomcat 启用gzip压缩
tags: Tomcat
categories: Tomcat
---
>Tomcat 启用gzip压缩

<!--more-->
## Tomcat启用gzip 压缩

**Gzip概念**

HTTP协议上的GZIP编码是一种用来改进WEB应用程序性能的技术。大流量的WEB站点常常使用GZIP[压缩技术](https://baike.baidu.com/item/压缩技术)来让用户感受更快的速度。这一般是指WWW服务器中安装的一个功能，当有人来访问这个服务器中的网站时，服务器中的这个功能就将网页内容压缩后传输到来访的电脑浏览器中显示出来。即：通过减小HTTP响应大小来减少响应时间。相对于普通的浏览过程HTML ,CSS,Javascript , Text ，它可以节省40%左右的流量。

这样传输就快了，效果就是你点击网址后会很快的显示出来。更为重要的是，它可以对动态生成的，包括CGI、PHP , JSP , ASP , Servlet,SHTML等输出的网页也能进行压缩，压缩效率也很高。当然这也会增加服务器的负载. 一般服务器中都安装有这个功能模块的



开启tomcat的Gzip只需修改server.xml配置文件，在Connector中添加下面几个参数即可

修改前：

```xml
 <Connector port="9980" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"/>

```

 修改后：    

```xml
 <Connector port="9980" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" 

               compression="on"
               compressionMinSize="2048"
               noCompressionUserAgents="gozilla, traviata" 
               useSendfile="false"
              
     compressableMimeType="text/html,text/xml,application/javascript,text/css,text/plain,application/json"/>
```

修改成功后，重启Tomcat

Chrome浏览器按F12 查看响应头有`Content-Encoding: gzip`即为成功



参数说明：

1. compression="on"

   开启Gzip压缩,默认为off

2. compressionMinSize="2048"

   大于2KB的文件才进行压缩，对资源压缩时会消耗一定的cpu性能，对2KB以上的资源才进行压缩是官方给出的建议，实际使用时可以根据需求在响应时间和cpu性能之间做取舍；

3. noCompressionUserAgents="gozilla, traviata"

   对于这两种浏览器，不进行压缩

4. useSendfile = "false"

   useSendfile属性默认为true, 会禁用任何可能的压缩, 改成false就好了（可不配置）

   tomcat默认设置是当数据大小达到48kb时，将启用文件传输（sendfile），所以我们想要压缩超过48kb的数据时必须将useSendfile设置为false

5. compressableMimeType="text/html,text/xml,application/javascript,text/css,text/plain,text/json"

   表明支持html、xml、js、css、json等文件格式的压缩



转载：https://www.cnblogs.com/sunonzj/p/12171502.html