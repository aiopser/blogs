---
title: hexo 个人博客搭建
tags: 
  - hexo 
  - next
categories: hexo
---
>hexo 博客安装

<!--more-->


转载：https://www.jianshu.com/p/eb002d35436c

### hexo 简介

Hexo是一个简单、快速、强大的基于 Github Pages 的博客发布工具，支持Markdown格式，有众多优秀插件和主题。

官网： [http://hexo.io](http://hexo.io/)
github: https://github.com/hexojs/hexo



### git 安装

安装略

git主要用于版本控制



### nodejs安装

1.下载并安装[NodeJS](https://modejs.org/en/)
这里选择最新的稳定版本v8.9.4安装过程可以一路next，不过推荐在选择安装路径时选择D盘，然后记住路径名，一般情况下是`D:\nodejs\`
 安装后输入`Win+R`输入`cmd`打开控制台输入以下代码：

```bash
node -v
npm -v
```

如果返回版本号即安装完成

2.nodejs配置

配置npm
我们要先配置npm的全局模块的存放路径以及cache的路径，最好在nodejs安装路径下建立"node_global"及"node_cache"两个文件夹

```
D:\nodejs\node_cache
D:\nodejs\node_global
```

启动cmd，输入

```cmd
npm config set prefix "D:\Program Files\nodejs\node_global"
npm config set cache "D:\Program Files\nodejs\node_cache"
```

配置环境变量:

此电脑--高级系统设置--高级--环境变量

USER的环境变量：

> 找到变量Path，点击编辑--新建 `D:\nodejs\node_global` -- 确认

系统的环境变量：

> 直接点击新建--变量名：NODE_PATH   
>
> 变量值：`D:\nodejs\node_global\node_modules`

测试是否配置完成，可以安装express来进行测试，打开cmd，输入

```cmd
npm install express -g
```

完成后在`node_global`目录下查看是否存在express目录即可。



### hexo 安装

打开git bash，为了避免出现错误后面的操作在git bash进行
 首先新建一个存放hexo文件的目录，例如在D盘根目录新建`hexo`文件夹，然后cd到该目录下，开始安装

```bash
$ cd D:hexo/    #换成你的目录
$ npm install -g hexo-cli  #安装hexo脚手架
$ hexo init     #Hexo自动在当前文件夹下下载搭建网站所需的所有文件
$ npm install   #安装依赖包

$ hexo g        #完整命令为hexo generate，生成静态文件
$ hexo s        #完整命令为hexo server，启动服务器，用来本地预览
```

用浏览器访问`http://localhost:4000/`，这时就可以看到了一个比较漂亮的博客了，这个是hexo的默认主题landscape



hexo+next系列文章：

https://blog.csdn.net/loze/article/details/94206726

