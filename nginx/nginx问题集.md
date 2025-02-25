# nginx问题集



## 访问网页403

查看error.log日志发现报错Permission denied，没有权限

查看nginx的启动用户，发现是nobody，而为是用root启动的

```
ps aux | grep "nginx: worker process" | awk'{print $1}'，
```

******解决方法：*进入nginx配置文件把user改为和启动用户一致

```
vim /etc/nginx/nginx.config

#################
user  root;
worker_processes  auto;
```



### 配置nginx.conf配置文件无效

检查是否更改的配置文件是/usr/local/nginx/conf目录下的