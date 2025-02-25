# centos7原码方式安装nginx

### 1.安装依赖

安装gcc环境

编译时依赖gcc环境

```
yum -y install gcc gcc-c++ autoconf automake make
```



安装 pcre

提供nginx支持重写功能

```
yum -y install pcre pcre-devel
```


安装zlib

zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩

```
yum -y install zlib zlib-devel make libtool
```


安装openssl

安全套接字层密码库，用于通信加密

```
yum -y install openssl openssl-devel
```



### 2.创建nginx用户以及用户组

```
groupadd nginx -g 666
useradd nginx -g ngisnx -u 666 -s /sbin/nologin -M  #创建nginx用户时加入nginx组，设置不能以shell形式登录nginx账号，不创建用户家目录
```







### 3.安装nginx

#### 1.下载tar包

```
wget http://nginx.org/download/nginx-1.6.2.tar.gz
```

解压

```
tar -zxvf nginx-1.6.2.tar.gz
```



#### 2.隐藏nginx版本号

```
vim nginx-1.6.2/src/core/nginx.h
```

```
/*
 * Copyright (C) Igor Sysoev
 * Copyright (C) Nginx, Inc.
 */


#ifndef _NGINX_H_INCLUDED_
#define _NGINX_H_INCLUDED_


#define nginx_version      1006002
#define NGINX_VERSION      "1.6.2"
#define NGINX_VER          "nginx/" NGINX_VERSION

#define NGINX_VAR          "NGINX"
#define NGX_OLDPID_EXT     ".oldbin"


#endif /* _NGINX_H_INCLUDED_ */
~                               
```

把NGINX_VERSION和NGINX_VAR修改掉

```
 /*
 * Copyright (C) Igor Sysoev
 * Copyright (C) Nginx, Inc.
 */


#ifndef _NGINX_H_INCLUDED_
#define _NGINX_H_INCLUDED_


#define nginx_version      1006002
#define NGINX_VERSION      "TWS"
#define NGINX_VER          "nginx/" NGINX_VERSION

#define NGINX_VAR          "TWS"
#define NGX_OLDPID_EXT     ".oldbin"


#endif /* _NGINX_H_INCLUDED_ */
```



#### 3.编译安装

```
cd nginx-1.6.2
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
make
make install
```

参数说明：

–prefix=/usr/local/nginx

#编译安装目录

–user=nginx

#所属用户nginx

–group=nginx

#所属组nginx

–with-http_stub_status_module

#该模块提供nginx的基本状态信息

–with-http_ssl_module



安装完后可以启动然后-V检查是否安装成功

```
root@localhost nginx-1.6.2]# /usr/local/nginx/sbin/nginx
[root@localhost nginx-1.6.2]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/TWS
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
```

或者可以这样检查配置文件是否正常

```
[root@localhost nginx-1.6.2]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```



#### 4.用curl查看nginx版本

```
curl -I 192.168.31.29 80
HTTP/1.1 200 OK
Server: nginx/TWS  #看不到nginx的版本，因为我们安装之前隐藏了
Date: Wed, 06 Dec 2023 09:05:51 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Wed, 06 Dec 2023 08:57:40 GMT
Connection: keep-alive
ETag: "65703784-264"
Accept-Ranges: bytes
```



### 4.配置全局使用nginx

```
ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/
```

HTTP)只输出HTTP-header，不获取内容(HTTP/FTP/FILE)。
用于HTTP服务时，获取页面的http头；
 （如：curl -I http://aiezu.com）
用于FTP/FILE时，将会获取文件大小、最后修改时间；
 （如：curl -I file://test.txt）