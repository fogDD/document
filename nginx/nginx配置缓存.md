# nginx配置缓存

 参考链接：https://blog.csdn.net/davidwkx/article/details/134191973

​	缓存能够存储请求的代理，以供未来再次使用，进而加速内容的提供。内容缓存可以缓存完整的响应，减少上游服务器的负载，避免了每次都为相同的请求重新运行计算和查询的麻烦。缓存可以提高性能并减少负载，这意味着可以用更少的资源更快地提供服务。NGINX 允许在NGINX 服务器的任何地方缓存内容。借助 NGINX 缓存，还可以在代理的服务器发生故障时被动地缓存并提供缓存的响应。缓存功能仅在 http 上下文中可用。
​       缓存由ngx_http_proxy_module模块实现。

### 缓存核心

无论什么缓存，都需要解决如下几个问题：

- 存哪儿
- 怎么存(存的结构，存的策略等)
- 缓存什么
- 如何刷新

[Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020)缓存示例（详解见后续）：

```cobol
http{
    ...
 
    # 缓存-存哪儿、怎么存(最大20G，3小时有效期，存储结构levels,按key-value存
    proxy_cache_path /home/nginxcache keys_zone=NCache:60m levels=1:2 inactive=3h max_size=20g;
 
    server {
        listen 80;
        server_name www.example.com;
 
        #使用NCache
        proxy_cache NCache;
        # 怎么存-缓存键，默认为“$scheme$proxy_host$request_uri”
        proxy_cache_key "$host$request_uri$cookie_JSESSIONID";
        # 存什么-GET、HEAD方法 （默认即可）
        #proxy_cache_methods GET HEAD;
        # 存什么-对应哪些响应存储、存储时间（必须定义）
        proxy_cache_valid                       200 206 304 301 302 10m;
        proxy_cache_valid                       any 1m;
        # 缓存更新-proxy_cache_background_update on;允许启动后台子请求以更新过期的缓存项，同时将过期的缓存响应返回到客户端。
        proxy_cache_background_update on;
 
        location / {
             # 其它-添加缓存命中状态，$upstream_cache_status  （调试用，非必须）
             add_header Nginx-Cache $upstream_cache_status;
             # 其它-针对代理服务器源码，忽略no-cache
             proxy_ignore_headers X-Accel-Expires Expires Cache-Control Set-Cookie;
 
             proxy_pass http://myserver;
             index       index.html index.htm;
        }
    }
}

```



### 存哪儿

Nginx的缓存数据保存在文件中，首先需定义缓存路径根目录，定义如下：

```cobol
    # 格式：proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [min_free=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
    # path ：缓存目录。缓存数据存储在该目录下的文件中，文件名是对缓存密钥应用MD5函数的结果，如：/data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c
    # levels：定义了缓存的层次结构级别：从1到3，每个级别都接受值1或2。缓存数据目录层次，最多3层，每层目录名长度为1或2
    # use_temp_path：是否使用单独的临时目录（缓存的响应首先写入临时文件，然后重命名该文件）。如果on，则使用proxy_temp_path指令定义的临时目录，此时存在copy效率；否则，临时目录就用缓存目录(path)，文件采用改名方式，不存在copy占时。
    # keys_zone：定义key共享内存区。所有key和有关数据的信息都存储在共享内存区域中，其名称和大小由keys_zone参数配置。一个兆字节的区域可以存储大约8000个key。
    # inactive：在该参数指定的时间内未访问的缓存数据将从缓存中删除。默认10分钟。
    # max_size：缓存区最大尺寸。当超过大小或没有足够的可用空间时，它会删除最近使用最少的数据。
    # min_free：保留最小可用空间量
    # manager_files、manager_threshold和manager_sleep：缓存数据缓存是迭代进行的。在一次迭代中，删除的项不超过manager_files定义数（默认为100个）。一次迭代的持续时间受manager_threshold参数的限制（默认为200毫秒）。在迭代之间，会进行由manager_sleep参数配置的暂停（默认为50毫秒）。
    # loader_files、loader_threshold和loader_sleep：将存储在文件系统上的缓存数据加载到缓存区域中，加载也是在迭代中完成的。在一次迭代中，加载项不超过loader_files（默认为100）。此外，一次迭代的持续时间受loader_threshold参数的限制（默认为200毫秒）。在迭代之间，会进行由loader_sleep参数配置的暂停（默认为50毫秒）。
    # purger：指示缓存清除程序是否会从磁盘中删除与通配符匹配的缓存项
    # purger_files、purger_threshold和purger_sleep：缓存清除程序在一次迭代中，清除项不超过purger_files（默认为10）。此外，一次迭代的持续时间受purger_threshold参数的限制（默认为50毫秒）。在迭代之间，会进行由loader_sleep参数配置的暂停（默认为50毫秒）。
    proxy_cache_path /data/nginx/cache keys_zone=NCache:60m levels=1:2 inactive=3h max_size=20g;
```

其实从参数说明可看出，proxy_cache_path也包含了存储策略。

proxy_cache_path 仅在http上下文中定义。



### 怎么存

​       实际上Nginx基于（key,value)模式存储，value就是响应内容，保存在文件中；文件具体位置由key来映射。示例如下：

    # 定义缓存key，默认为“$scheme$proxy_host$request_uri”，可自定义
    proxy_cache_key "$host$request_uri $cookie_JSESSIONID";
     
    # 使用几次才进行缓存，默认为1
    proxy_cache_min_uses 1;
  选择一个好的proxy_cache_key非常重要，其中离不开对应用的了解。对html纯静态内容选择缓存键通常非常简单，只需 URI 便可。但如果是为动态的内容选择缓存键，就要求对用户与应用的交互方式以及用户体验之间的差异程度有着更深刻的认识。



### 缓存什么

缓存内容肯定是Nginx响应前端的信息。使用示例如下：

        # proxy_cache 定义当前上下文Key使用哪个缓存共享内存区域（在proxy_cache_path 中定义的）。同一区域可以在多个地方使用。如果是proxy_cache off，禁用从以前的配置级别继承的缓存
        proxy_cache NCache;
     
        # 基于URL方法定义哪些URL缓存，默认GET、HEAD方法
        #proxy_cache_methods GET HEAD;
     
        # proxy_cache_valid [code ...] time;  基于响应代码设置缓存数据的时间。实际上只有定义proxy_cache_valid，缓存才表示被真正使用
        proxy_cache_valid  200 206 304 301 302 10d;
        proxy_cache_valid  any 1m;
注意：实际上只有定义proxy_cache_valid，缓存才表示被真正使用

 可在http, server, location上下文中使用缓存 。



### 缓存刷新

​         在代理模式下，Nginx缓存网站的静态内容，减轻源服务器的负载，加快网站的访问速度。然而，当网站内容发生变化时，Nginx并不会立即更新缓存，导致用户看到的是旧的页面内容。Nginx由如下几种解决方式。

方式1-过期刷新
配置Nginx在缓存过期后主动去源服务器请求新的内容。配置如下：

```
# 缓存更新-proxy_cache_background_update on | off;默认off。on-允许启动后台子请求以更新过期的缓存项，同时将过期的缓存响应返回到客户端。
proxy_cache_background_update on;
```

 		即使proxy_cache_background_update on，Nginx在缓存过期后也不会主动去源服务器请求新的内容，而是等下次请求访问，才向服务器重新获取需缓存内容并再次缓存。     

​		该种方式虽然能更新缓存，但是只有等缓存过期才有效。

#### 方式2-删除缓存文件

​     	暴力删除缓存文件（即proxy_cache_path 定义路径下的文件），即删除了所有的缓存，下次被访问时再缓存。

​     	这种方式虽然非常有效，但只适合应用使用初期。对于成熟的应用，可能导致前端访问出现比较大的抖动。

#### 方式3-指定更新

针对更新的URL更新缓存，示例如下：

```
http{
    ...

    # purger=on指示缓存清除程序是否会从磁盘中删除与通配符匹配的缓存项；默认为off。注意：purger_files、purger_threshold和purger_sleep三个参数是否满足要求。

​    proxy_cache_path /data/nginx/cache keys_zone=NCache:60m levels=1:2 inactive=3h max_size=20g purger=on;
​     

    # 请求方法是PURGE，则表示清除缓存

​    map $request_method $purge_method {
​        PURGE   1;
​        default 0;
​    }
​     
​    server {
​        ...
​        location / {
​            ...

            # 清除缓存

​            proxy_cache_purge $purge_method;
​        }    
​    }

}


```

然后从过curl发起清除命令（可用*匹配）：

curl -X PURGE http://www.example.com/main.js
curl -X PURGE http://www.example.com/path/b*.css
注意：指令proxy_cache_purge在模块nginx-module-cache-purge中，商业版才支持。

最佳方法
       最佳缓存刷新方法就是不用Nginx缓存刷新，而是在应用开发时解决，可通过前端加版本号工具（如grunt-cachebuster），给页面请求的每个更新过的资源加上新版本号，这样每次更新的文件都没有被缓存过，就不存在缓存刷新问题。