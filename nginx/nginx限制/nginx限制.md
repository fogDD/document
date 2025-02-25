# nginx限制

### 一、nginx限制ip不可访问

**nginx配置如下：**

```
    server {
      listen      80;
      server_name 192.168.31.100;
      server_name 192.168.31.200;
      ###如果访问的ip地址为192.168.9.115,则返回403###
      if ($remote_addr = 192.168.31.214) {
        return 403;
      }

```

**验证：**

ip为192.168.31.214的地址访问显示403

![WeChat03bb686ef910a78f3c7f274247ce9b67](/Users/zhouqiting/Desktop/document/nginx/nginx限制/WeChat03bb686ef910a78f3c7f274247ce9b67.jpg)



### 二、使用allow、deny指令限制ip访问

**nginx配置如下：**

```
      location / {
        # 允许特定的ip地址访问
        allow 192.168.29.0/24;
        # 拒绝其他ip地址访问
        deny all;
      }
```

**配置禁止192.168.29.0以外的ip地址访问**

ip为192.168.29.0以内的设备访问会显示403

验证：

![](/Users/zhouqiting/Desktop/document/nginx/nginx限制/WechatIMG1.jpg)

说明：

1,注意在使用指令时，

如果最后不添加deny all,则可能会允许上面列出ip之外的其他ip均可访问

因为默认是allow all的，

如果只想禁止指定的ip访问，只添加deny x.x.x.x 指令即可

 

2,如果安全规则想针对全站，放在server段内比较安全，

  如果只针对指定的目录做限制,要注意检查location的匹配



### 三、nginx限制浏览器访问

nginx限制google浏览器访问

**nginx配置如下：**

```
   server {
      listen      80;
      server_name 192.168.31.100;
      server_name 192.168.31.200;
      ###如果访问的ip地址为192.168.9.115,则返回403###
      #if ($remote_addr = 192.168.31.214) {
      #  return 403;
      #}

      ###如果访问的浏览器是谷歌，则返回500###
      if ($http_user_agent ~ Chrome ){
        return 500;
      }
```

**浏览器访问验证：**

![WechatIMG2](/Users/zhouqiting/Desktop/document/nginx/nginx限制/WechatIMG2.jpg)