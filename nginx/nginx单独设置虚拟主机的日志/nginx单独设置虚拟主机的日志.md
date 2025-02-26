# nginx单独设置虚拟主机的日志

### 1. nginx.conf总配置文件配置

原配置文件

```
user  www;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$document_uri $remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    #access_log  off;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```



修改后的配置文件

因为我这里虚拟主机的配置文件都include了，所以主要的配置都在虚拟主机的conf文件中

```
user  www;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    #access_log  off;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```





### 2.虚拟主机配置文件

进入虚拟主机配置文件夹

```
[root@huoying ~]# cd /etc/nginx/conf.d/
```

虚拟主机配置文件

log_format需要和http同级，access_log目录位置可以写在server{}里边

```
    log_format  main  '$document_uri $remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

server{
  access_log  /var/log/nginx/huoying.linux0224.cc.log  main;
  listen 80;
  server_name huoying.linux0224.cc;
  charset utf-8;
  location  /  {
    root /www/huoying/;
    index index.html;
  }
}
```



配置第二个虚拟主机

虚拟主机配置文件

main变量不能重复，所以这边写main2，log日志名字也要单独设置，因为是以虚拟主机为单位

```
    log_format  main2 '$document_uri $remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

server{
  access_log  /var/log/nginx/juice.log  main2;
  listen 80;
  server_name juiceworld.cc;
  charset utf-8;

  location / {
    root /www/juiceworld/;
    index index.html;
  }
}
```



保存完后记得nginx -t配置一下，没问题即可重启





### 3.剩下的虚拟主机统计记录

剩余其他的虚拟主机日志，全部统一记录到 /var/log/nginx/all-server-accesss.log

nginx.conf主配置文件如下，main3不能和上边两个虚拟主机配置文件中日志的变量相同，否则会失效

```
user  www;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main3  '$document_uri  $remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/all-server-accesss.log  main3;
    #access_log  off;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```







### 3.访问测试

进入存放log目录查看是否有对应的log文件

![image-20240410111653692](/Users/zhouqiting/Desktop/document/nginx/nginx单独设置虚拟主机的日志/image-20240410111653692.png)

可以看到刚设置的两个虚拟主机的配置日志已经生成了



同时监控两个虚拟主机的配置文件，访问网站测试

```
[root@huoying ~]# cd /var/log/nginx/
[root@huoying nginx]# tail -f ./*
[root@huoying conf.d]# curl juiceworld.cc
[root@huoying conf.d]# curl huoying.linux0224.cc
[root@huoying conf.d]# curl 192.168.31.29:81
[root@huoying conf.d]# curl 192.168.31.29:82
```

![image-20240410160516520](/Users/zhouqiting/Desktop/document/nginx/nginx单独设置虚拟主机的日志/image-20240410160516520.png)

可以看到我们curl不同的虚拟主机，日志会生成在不同的虚拟主机文件上，剩余的日志即生成all-server-accesss.log中，成功！