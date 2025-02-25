# nginx配置https

在配置https之前请确保下面的步骤已经完成

1. 服务器已经安装nginx并且通过http可以正常访问

2. 拥有ssl证书，没有的可以去阿里购买或者免费申请一年

3. 检查nginx是否安装了ssl模块

   nginx -V以下状态的就是安装了ssl模块，configure配置中含有ssl模块

   ```
   [root@host1 objs]# nginx -V
   nginx version: nginx/1.22.1
   built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
   built with OpenSSL 1.0.2k-fips  26 Jan 2017
   TLS SNI support enabled
   configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
   ```

   

### 一、nginx安装ssl模块

进入到nginx解压目录，注意不是nginx的安装目录，执行

```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make
```

完成上诉操作，`/root/nginx-1.22.1/objs/`目录下会出现nginx文件，接下来要把新的nginx文件替换到已安装nginx的目录下的nginx文件，**替换之前可以先备份nginx文件**

```
nginx -s stop
cp ~/nginx-1.22.1/objs/nginx /usr/local/sbin/nginx
```

成功之后，输入`nginx -V`查看

```
[root@host1 objs]# nginx -V
nginx version: nginx/1.22.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

