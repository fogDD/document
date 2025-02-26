# [CentOS7安装及配置 Zabbix Server全步骤，超详细教程]

https://blog.csdn.net/weixin_50085977/article/details/123364688

https://www.cnblogs.com/linnuo/p/15933012.html

## 一、监控软件的功能

通过一个友好的界面进行浏览整个网站所有的服务器状态
可以在 Web 前端方便的查看监控数据
可以回溯寻找事故发生时系统的问题和报警情况
使用监控系统查看服务器状态以及网站流量指标，利用监控系统的数据去了解上线发布的结果，和网站的健康状态

## 二、zabbix定义

zabbix 是一个基于 Web 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案
zabbix 能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题
zabbix 由 2 部分构成，zabbix server 与可选组件 zabbix agent。通过 C/S 模式采集数据，通过 B/S 模式在 Web 端展示和配置
zabbix server 可以通过 SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能， 它可以运行在 Linux 等平台上
zabbix agent 需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU 等信息的收集

## 三、zabbix监控原理

zabbix agent安装在被监控的主机上，zabbix agent负责定期收集客户端本地各项数据，并发送至 zabbix server 端，zabbix server 收到数据后，将数据存储到数据库中，用户基于 Zabbix WEB 可以看到数据在前端展现图像。当 zabbix 监控某个具体的项目， 该项目会设置一个触发器阈值，当被监控的指标超过该触发器设定的阈值，会进行一些必要的动作，动作包括：发送信息（邮件、微信、短信）、发送命令（shell 命令、reboot、restart、install 等

## 四、zabbix的五个程序

zabbix server：zabbix 服务端守护进程，其中 zabbix_agent、zabbix_get、zabbix_sender、zabbix_proxy 的数据最终都提交给 zabbix server
zabbix agent：客户端守护进程，负责收集客户端数据，例如:收集 CPU 负载、内存、硬盘使用情况等
zabbix proxy：zabbix 分布式代理守护进程，通常大于 500 台主机，需要进行分布式监控架构部署
zabbix get：zabbix 数据接收工具，单独使用的命令，通常在server或者 proxy 端执行获取远程客户端信息的命令
zabbix sender：zabbix 数据发送工具，用户发送数据给 server 或 proxy 端，通常用户耗时比较长的检查

## 五、安装zabbix5.0

### 5.1 部署zabbix服务端

#### 5.1.1 环境准备

```
systemctl disable --now firewalld
systemctl stop firewalld.service
setenforce 0
hostnamectl set-hostname 

su
```

#### 5.1.2 获取zabbix的下载源

```
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

vi.zabbix.repo
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```

#### 5.1.3 修改zabbix-front前端源

```
vim zabbix.repo
......
[zabbix-frontend]
......
enabled=1          #开启安装源
......
 
yum clean all
yum makecache fast
```

![1660098541977](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660098541977.png)

#### 5.1.4 安装zabbix组件

```
yum install -y zabbix-server-mysql zabbix-agent
yum install -y centos-release-scl 
```

#### 5.1.5 安装zabbix前端环境到scl环境下

```
yum install -y zabbix-web-mysql-scl zabbix-apache-conf-scl
```

#### 5.1.6 安装zabbix所需的数据库

```
yum install -y mariadb-server mariadb
 
systemctl enable --now mariadb
 
mysql_secure_installation         #初始化数据库，并设置密码，如123456
```

![img](https://img-blog.csdnimg.cn/img_convert/64bdae058d518370a92d0b7d81a549fe.png)

![img](https://img-blog.csdnimg.cn/img_convert/64bdae058d518370a92d0b7d81a549fe.png)

#### 5.1.7 添加数据库用户，以及zabbix所需的数据库信息

```
mysql -u root -p264196
 
CREATE DATABASE zabbix character set utf8 collate utf8_bin;
GRANT all ON zabbix.* TO 'zabbix'@'%' IDENTIFIED BY 'zabbix';
flush privileges;
```

![1660098824351](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660098824351.png)

#### 5.1.8 导入数据库信息

```
rpm -ql zabbix-server-mysql 		#查询 sql 文件的位置
 
zcat /usr/share/doc/zabbix-server-mysql-5.0.15/create.sql.gz | mysql -uroot -p123456 zabbix
```

![1660098934715](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660098934715.png)

![img](https://img-blog.csdnimg.cn/img_convert/d428b1aa9c99382e43e293b1820b058b.png)

#### 5.1.9 修改zabbix server配置文件，修改数据库的密码

```
vim /etc/zabbix/zabbix_server.conf 
......
DBPassword=zabbix	
```

![1660099155369](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099155369.png)

#### 5.1.10 修改zabbix的php配置文件

```
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
......
php_value[date.timezone] = Asia/Shanghai		#24行，取消注释，修改时区
```

#### ![1660099290110](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099290110.png)5.1.11 启动zabbix相关服务

```
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

#### 5.1.12 浏览器访问

```
http://192.168.100.10/zabbix
点击下一步，设置数据库的密码 zabbix
安装完成后，默认的登录账号和密码为：Admin/zabbix
设置文件界面：点击左边菜单栏的【User settings】，【Language】选择 Chinese(zh_CN)，再点击 Update 更新。
 
//解决 zabbix-server Web页面中文乱码问题
yum install -y wqy-microhei-fonts
 
\cp -f /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
```

![1660099603831](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099603831.png)!1660099509043](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099509043.png)

![1660099575283](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099575283.png)

![1660099742457](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660099742457.png)

### 5.2 部署zabbix客户端

#### 5.2 部署zabbix客户端

```
systemctl disable --now firewalld
setenforce 0
hostnamectl set-hostnas me zbx-agent01
su
```

#### h5.2.2 服务端和客户端都配置时间同步

```
yum install -y ntpdate
ntpdate -u ntp.aliyun.com
```

![img](https://img-blog.csdnimg.cn/img_convert/061ae8d29bbdc7ff55dbcf89788be1da.png)

#### 5.2.3 客户端配置时区，与服务器保持一致

```
mv /etc/localtime{,.bak}
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 
date
```

#### 5.2.4 设置zabbix的下载源，安装zabbix-agent2

```
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

vi.zabbix.repo
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```

![1660100741079](C:\Users\星冥轩\AppData\Roaming\Typora\typora-user-images\1660100741079.png)

#### 5.2.5 修改agent2配置文件并启动zabbix-agent2

```
vim /etc/zabbix/zabbix_agent2.conf
......
Server=192.168.100.10			#80行，指定 zabbix 服务端的 IP 地址
ServerActive=192.168.100.20		#120行，指定 zabbix 服务端的 IP 地址
Hostname=zbx-agent01			#131行，指定当前 zabbix 客户端的主机名
 
systemctl start zabbix-agent2
systemctl enable zabbix-agent2
 
netstat -natp | grep zabbix
#客户端对接端口10050
```

#### 5.2.6 在服务端验证zabbix-agent2的连通性

```
yum install -y zabbix-get				#安装 zabbix 主动获取数据的命令
 
zabbix_get -s '192.168.100.20' -p 10050 -k 'agent.ping'
1
 
zabbix_get -s '192.168.100.20' -p 10050 -k 'system.hostname'
zbx-agent01
```

#### 5.2.7 在Web页面中添加agent主机

```
点击左边菜单栏【配置】中的【主机】，点击【创建主机】
【主机名称】设置成 zbx-agent01
【可见的名称】设置成 zbx-agent01-192.168.100.20
【群组】选择 Linux server
【Interfaces】的【IP地址】设置成 192.168.100.20
 
再点击上方菜单栏【模板】
【Link new tamplates】搜索 Linux ，选择 Template OS Linux by Zabbix agent
点击 【添加】
```

![img](https://img-blog.csdnimg.cn/f8745c94c5ec4d91887db8b41130f39f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54as5aSc56eD5aS05bCP546L5a2Q,size_20,color_FFFFFF,t_70,g_se,x_16)

![img](https://img-blog.csdnimg.cn/fddb95ded8e64b23bbbce2ac4e677f10.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54as5aSc56eD5aS05bCP546L5a2Q,size_20,color_FFFFFF,t_70,g_se,x_16)

![img](https://img-blog.csdnimg.cn/fddb95ded8e64b23bbbce2ac4e677f10.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54as5aSc56eD5aS05bCP546L5a2Q,size_20,color_FFFFFF,t_70,g_se,x_16)

## 六 自定义监控内容