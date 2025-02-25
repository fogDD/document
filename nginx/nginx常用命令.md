### nginx常用命令

启动nginx

```
/usr/local/nginx/sbin/nginx
```



从容停止

```
kill -QUIT 【Nginx主进程号】 
```



快速停止

```
kill -TERM 【Nginx主进程号】
```

或

```
pkill nginx
```

或

```
/usr/local/nginx/sbin/nginx -s stop
```



检查nginx配置文件是否正确

```
[root@localhost ~]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```



平滑启动nginx

```
/usr/local/nginx/sbin/nginx -s reload
```

或

```
kill -HUP 【Nginx主进程号】、
```

其中进程文件路径在配置文件 nginx。conf中可以找到。

平滑启动的意思是在不停止ginx的情况下，重启ginx， 重新加载

配置文件，启动新的工作线程，完美停止旧的工作线程。



nginx查看版本

小v查看版本，大V查看详细信息

```
[root@localhost ~]# /usr/local/nginx/sbin/nginx -v
nginx version: nginx/TWS
[root@localhost ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/TWS
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
```

