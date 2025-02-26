## 配置ntp服务

### 1.安装NTP包

需要先安装ntp的依赖包ntpdate和autogen-libopts

```
[root@localhost ~]#  rpm -qa | grep ntp
ntpdate-4.2.6p5-1.el6.x86_64
fontpackages-filesystem-1.41-1.1.el6.noarch
ntp-4.2.6p5-1.el6.x86_64
```



## 2.配置ntp服务

### 	2-1.配置主节点

```
vi /etc/ntp.conf

restrict **.**.**.**(ntp服务器ip) nomodify notrap nopeer noquery


restrict **.**.**.**(网关) mask 255.255.255.0 nomodify notrap 

server 127.127.1.0
Fudge 127.127.1.0 stratum 10
```

语法为： restrict IP地址 mask 子网掩码 参数

其中IP地址也可以是default ，default 就是指所有的IP

参数有以下几个：

ignore ：关闭所有的 NTP 联机服务

nomodify：客户端不能更改服务端的时间参数，但是客户端可以通过服务端进行网络校时。

notrust ：客户端除非通过认证，否则该客户端来源将被视为不信任子网

noquery ：不提供客户端的时间查询：用户端不能使用ntpq，ntpc等命令来查询ntp服务器

notrap ：不提供trap远端登陆：拒绝为匹配的主机提供模式 6 控制消息陷阱服务。陷阱服务是 ntpdq 控制消息协议的子系统，用于远程事件日志记录程序。

nopeer ：用于阻止主机尝试与服务器对等，并允许欺诈性服务器控制时钟

kod ： 访问违规时发送 KoD 包。

restrict -6 表示IPV6地址的权限设置。





### 2-2配置分节点

```
vi /etc/ntp.conf

server **.**.**.**(主节点的ip) prefer
```

其中host是上层NTP服务器的IP地址或域名，随后所跟的参数解释如下所示：

◆ key： 表示所有发往服务器的报文包含有秘钥加密的认证信息，n是32位的整数，表示秘钥号。

◆ version： 表示发往上层服务器的报文使用的版本号，n默认是3，可以是1或者2。

◆ prefer： 如果有多个server选项，具有该参数的服务器有限使用。

◆ mode： 指定数据报文mode字段的值。

◆ minpoll： 指定与查询该服务器的最小时间间隔为2的n次方秒，n默认为6，范围为4-14。

◆ maxpoll： 指定与查询该服务器的最大时间间隔为2的n次方秒，n默认为10，范围为4-14。

◆ iburst： 当初始同步请求时，采用突发方式接连发送8个报文，时间间隔为2秒



## 3.配置完成

### 	3-1.重启服务

```
systemctl restart ntpd
```

查看ntp服务器有无和上层ntp连通

```
ntpstat
ntpq -p
```

出现server(主节点)表示同步成功，需要等待5-10分钟才能与/etc/ntp.conf中配置的标准时间进行同步。



## 4.注意

### 	4-1.防火墙屏蔽ntp端口

ntp服务器默认端口是123，如果防火墙是开启状态，在一些操作可能会出现错误，所以要记住关闭防火墙。

```
iptables -I INPUT -p tcp --dport 123 -j ACCEPT
iptables -I INPUT -p udp --dport 123 -j ACCEPT
```

设置完记得重启ntp服务