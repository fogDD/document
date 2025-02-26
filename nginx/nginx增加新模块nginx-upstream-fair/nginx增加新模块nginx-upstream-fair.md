# nginx增加新模块nginx-upstream-fair

把nginx-upstream-fair模块下载好，传到服务器上解压



#### 下载一个相同版本的nginx

#### 查看当前版本的nginx

```
[root@www ~]# nginx -V
nginx version: nginx/1.22.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
configure arguments:
```

#### 下载相同的nginx版本并解压，添加新模组目录

```
tar -zxvf nginx-1.22.1.tar.gz
cd nginx-1.22.1
./configure --add-module=/root/nginx-upstream-fair-master
```

--add-module=/root/nginx-upstream-fair-master是你nginx-upstream-fair-master模块的安装目录

#### 编译

```
make
```

![WeChat1959fe02cf6c1655d7370434060989d7](/Users/zhouqiting/Desktop/document/nginx/nginx增加新模块nginx-upstream-fair/WeChat1959fe02cf6c1655d7370434060989d7.jpg)

编译可能会出现端口配置问题，ngx_http_upstream_srv_conf_t结构中缺少default_port

#### 解决方案：

在Nginx的源码中 src/http/ngx_http_upstream.h,找到`ngx_http_upstream_srv_conf_s`，在模块中添加添加default_port属性

```
in_port_t	   default_port
```

![WeChat26a10df79daf37aed0020ca187443d9e](/Users/zhouqiting/Desktop/document/nginx/nginx增加新模块nginx-upstream-fair/WeChat26a10df79daf37aed0020ca187443d9e.jpg)

####  将目前正在使用的nginx sbin目录下的nginx进行备份

```
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginxbak
```

#### 将安装目录下的objs中的nginx拷贝到现在使用的nginx中的sbin目录

```
cp objs/nginx /usr/local/nginx/sbin/
```

#### 更新nginx

```
make upgrade
nginx -s reload
```

