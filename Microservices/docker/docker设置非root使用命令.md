# docker设置普通用户使用命令

[root@node1 ~]# vim /etc/sudoers

jialideng  ALL=(ALL) NOPASSWD: /usr/local/bin/docker

![](/Users/zhouqiting/Desktop/document/微服务/docker/docker设置非root用户使用命令/WeChatb4116838ea8e9d655de584bc797b617f.jpg)

检查是否成功

![](/Users/zhouqiting/Desktop/document/微服务/docker/docker设置非root用户使用命令/WeChat25cbf2e3049fc00b4ffe713c9d4b1e0e.jpg)