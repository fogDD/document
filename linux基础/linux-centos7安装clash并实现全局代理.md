# linux-centos7安装clash并实现全局代理

## 1.把安装包传到本地并设置执行权限

```
[root@k8s-master home]# gunzip clash-linux-amd64-v1.18.0.gz 
[root@k8s-master home]# ll
-rw-r--r-- 1 root root  9904128 Jun  4 03:01 clash-linux-amd64-v1.18.0
[root@k8s-master home]# chmod +x clash-linux-amd64-v1.18.0 
[root@k8s-master home]# mv clash-linux-amd64-v1.18.0 clash-v1.13.0

[root@k8s-master home]# ./clash-v1.13.0 
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download              
INFO[0000] inbound mixed://127.0.0.1:7890 create success. 
```

第一行是运行clash，记得要在文件名前面加上`./`才行
第二行是clash的提示，说“没有配置文件，已为您创建完成”，但是文件是空的，无法使用
第三行也是，说“没有找到MMDB文件，开始下载”，但是会下载失败

下面我们先开始解决这两个问题



## 2.解决config.yaml和MMDB问题

### 2.1配置config.yaml

config.yaml为clash的代理规则和clash的一些其他设置。代理规则不需要我们自己编写，通过订阅地址直接下载即可

此处将订阅链接粘贴进双引号中间。注意不要删除双引号，不要删除空格

```
wget -O ~/.config/clash/config.yaml "订阅链接"
```

```
[root@k8s-master ~]# wget -O ~/.config/clash/config.yaml "https://msub.fengchiyx.xyz/api/v1/client/subscribe?token=acd5ceaad7a6e5f487b61e0755247ce4"
--2025-06-04 03:12:07--  https://msub.fengchiyx.xyz/api/v1/client/subscribe?token=acd5ceaad7a6e5f487b61e0755247ce4
Resolving msub.fengchiyx.xyz (msub.fengchiyx.xyz)... 160.202.245.83
Connecting to msub.fengchiyx.xyz (msub.fengchiyx.xyz)|160.202.245.83|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘/root/.config/clash/config.yaml’

    [ <=>                                                                                                                                    ] 16,292      --.-K/s   in 0.01s   

2025-06-04 03:12:08 (1.33 MB/s) - ‘/root/.config/clash/config.yaml’ saved [16292]
```

```
我这里是把笔记本的clash软件找到配置文件的yaml替换了了服务器的clash的config.yaml文件
```

至此config.yaml搞定



### 2.2配置MMDB文件

Country.mmdb为全球IP库，可以实现各个国家的IP信息解析和地理定位，没有这个文件clash是无法运行的。
但目前版本的clash有点问题，不会自动生成MMDB文件，所以需要使用命令行下载
直接在终端运行此代码即可

```
[root@k8s-master home]# cp Country.mmdb ~/.config/clash/Country.mmdb
```

我这边是直接把Country.mmdb文件下载到本地直接放入clash目录，因为用wget下载Country.mmdb文件需要翻墙



## 3.正式启动Clash

```
[root@k8s-master home]# ./clash-v1.13.0 
INFO[0000] Start initial compatible provider 磁力下载       
INFO[0000] Start initial compatible provider 谷歌服务       
INFO[0000] Start initial compatible provider TikTok     
INFO[0000] Start initial compatible provider WhatsApp   
INFO[0000] Start initial compatible provider Netflix    
INFO[0000] Start initial compatible provider Steam      
INFO[0000] Start initial compatible provider 苹果服务       
INFO[0000] Start initial compatible provider 境内使用       
INFO[0000] Start initial compatible provider 抖音         
INFO[0000] Start initial compatible provider 海外使用       
INFO[0000] Start initial compatible provider 故障转移       
INFO[0000] Start initial compatible provider 自动选择       
INFO[0000] Start initial compatible provider 哔哩哔哩       
INFO[0000] Start initial compatible provider Telegram   
INFO[0000] Start initial compatible provider Twitter    
INFO[0000] Start initial compatible provider Copilot    
INFO[0000] Start initial compatible provider 节点选择       
INFO[0000] Start initial compatible provider ChatGPT    
INFO[0000] Start initial compatible provider 微软服务       
INFO[0000] inbound mixed://127.0.0.1:7890 create success. 
INFO[0000] DNS server listening at: 127.0.0.1:1053      
```

`INFO[0000] inbound mixed://127.0.0.1:7890 create success. `则代表已经开启了http(含https)和socks代理，只要服务器内有软件流量通过7890这个端口，流量都将进入clash从而被代理。（但有些不支持设置，后面会说如何使用全局代理）



## 4.后台运行Clash

如果按照上面的方法运行clash的话，一旦我们关闭了终端，那么clash也会一并关闭。所以我们需要设置一下后台运行clash
在`/etc/systemd/system/`目录新建一个`clash.service`文件，并且直接进入vim

```
[root@k8s-master clash]# vim /etc/systemd/system/clash.service
```

```
[Unit]
Description=Clash service
After=network.target

[Service]
Type=simple
User=root
ExecStart=这里写你的clash运行的绝对路径（本文中的路径是/usr/local/clash/clash-v1.13.0）
Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```

```
[root@k8s-master clash]# systemctl start clash
```

查看服务是否运行正常

```
[root@k8s-master clash]# systemctl status clash
● clash.service - Clash service
   Loaded: loaded (/etc/systemd/system/clash.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2025-06-04 03:55:46 EDT; 28s ago
 Main PID: 15287 (clash-v1.13.0)
    Tasks: 13
   Memory: 8.9M
   CGroup: /system.slice/clash.service
           └─15287 /home/clash/clash-v1.13.0

Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider Twitter"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider Netflix"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider 节点选择"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider ChatGPT"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider 谷歌服务"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider 磁力下载"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider Telegram"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="Start initial compatible provider 境内使用"
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="inbound mixed://127.0.0.1:7890 create success."
Jun 04 03:55:46 k8s-master clash-v1.13.0[15287]: time="2025-06-04T03:55:46-04:00" level=info msg="DNS server listening at: 127.0.0.1:1053"
```



### systemctl常用命令

`systemctl status clash` 查看clash服务
`systemctl start clash` 启动clash服务
`systemctl stop clash` 停止clash服务
`systemctl restart clash` 重启clash服务
`systemctl enable clash` 设置开机自启clash服务
`systemctl daemon-reload` 如果修改了`clash.service`文件，需要此命令来重载被修改的服务文件



## 5.测试代理是否正常

根据我们之前前台运行可得知，默认是监控了自己的7890端口

```
INFO[0000] inbound mixed://127.0.0.1:7890 create success. 
```

那么现在，使用`curl`来向谷歌发送一个请求看能否正常返回数据

```
[root@k8s-master clash]# c
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com.hk/url?sa=p&amp;hl=zh-CN&amp;pref=hkredirect&amp;pval=yes&amp;q=http://www.google.com.hk/&amp;ust=1749023849850989&amp;usg=AOvVaw1OC3YVfXQ-AG7CBUAOuivk">here</A>.
</BODY></HTML>
```

Google成功返回数据，代表7890端口代理正常，clash运行正常





## 6.配置全局代理

设置全局代理需要在`/etc/profile`文件最后添加几行命令
进入`/etc/profile`文件

```
[root@k8s-master clash]# vim /etc/profile
```

在文件的最后加入两行proxy代理命令

```
export http_proxy=127.0.0.1:7890
export https_proxy=127.0.0.1:7890
```

加入后保存文件、应用配置文件、重启服务、测试

```
[root@k8s-master clash]# source /etc/profile
[root@k8s-master clash]# systemctl restart clash
[root@k8s-master clash]# 
[root@k8s-master clash]# systemctl status clash
● clash.service - Clash service
   Loaded: loaded (/etc/systemd/system/clash.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2025-06-04 04:01:18 EDT; 18s ago
 Main PID: 19335 (clash-v1.13.0)
    Tasks: 13
   Memory: 8.9M
   CGroup: /system.slice/clash.service
           └─19335 /home/clash/clash-v1.13.0

Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider WhatsApp"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider Twitter"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider 磁力下载"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider 海外使用"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider 故障转移"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider 节点选择"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider 抖音"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="Start initial compatible provider Telegram"
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="inbound mixed://127.0.0.1:7890 create success."
Jun 04 04:01:18 k8s-master clash-v1.13.0[19335]: time="2025-06-04T04:01:18-04:00" level=info msg="DNS server listening at: 127.0.0.1:1053"
[root@k8s-master clash]# curl  www.google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com.hk/url?sa=p&amp;hl=zh-CN&amp;pref=hkredirect&amp;pval=yes&amp;q=http://www.google.com.hk/&amp;ust=1749024148571613&amp;usg=AOvVaw2YznpiPOw0oEMlEyj5MCeR">here</A>.
</BODY></HTML>
```



## 7.测试helm拉取bitnami/redis-21.0.3包

```
[root@k8s-master clash]# helm pull bitnami/redis
total 9344
-rw-r--r-- 1 root root   16384 Jun  4 03:04 cache.db
-rw-r--r-- 1 root root  221006 Jun  4 03:47 config.yaml
-rw-r--r-- 1 root root 9215458 Jun  4 03:36 Country.mmdb
-rw-r--r-- 1 root root  111483 Jun  4 04:04 redis-21.0.3.tgz
[root@k8s-master clash]# 
```

