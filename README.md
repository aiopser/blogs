同步hexo博客

使用场景：家里，公司

操作：
同步了两边的环境都一致并且都可以使用 hexo clean && hexo g -d 部署文章后

每次`文章发布`完成后执行如下命令：
```bash
$ cd repos/sync_blog/
$ git add .
$ git commit -m "commit msg."
$ git push origin master
```

`写文章`之前执行如下命令：
```bash
$ cd repos/sync_blog/
$ git pull origin master
```



