# nginx配置gzip

### 1.配置文件

```
http {
    ...
    ...
###开启gzip限流
    #开启gzip
    gzip  on;
    #gzip最小压缩文件大小，超过这个大小即启用压缩
    gzip_min_length 1k;
    #指定压缩级别范围为1到9，值越大压缩程度越大；随之CPU消耗也越大，CPU消耗对服务器性能也会有影响
    gzip_comp_level 1;
    #如果发送的响应被gzip压缩，则在响应头部加上Vary：Accept-Encoding“，以通知缓存服务器响应内容可能以压缩或非压缩形式存在
    gzip_vary on;
    #指定不使用gzip压缩的User-Agent 禁用IE6gzip
    gzip_disable "MSIE [1-6]\.";
    #指定用于gzip压缩的内存缓冲区大小
    gzip_buffers 4 16k;
    #设置进行gzip压缩的HTTP协议版本。如果nginx使用了proxy_pass反向代理，则默认使用http1.0版本的代理传输
    gzip_http_version 1.0;
    #指定要压缩的MIME类型
    gzip_types       text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
```

设置完毕后重新应用nginx

![WeChat7cf0aeb8e3e3cd46883cb4e49527d7fb](/Users/zhouqiting/Desktop/document/nginx/nginx配置gzip/WeChat7cf0aeb8e3e3cd46883cb4e49527d7fb.jpg)

浏览器访问nginx网页，F12打开后台检查，检查响应测试是否成功应用gzip